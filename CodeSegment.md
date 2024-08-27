# 大文件分片上传

### **前端部分**

**组件**

```vue
<template>
  <a-upload :beforeUpload="beforeUploadDirectory" :showUploadList="false" accept=".zip">
    <a-button type="primary" :loading="uploading">
      <a-upload-outlined />
      导入拍摄点
    </a-button>
  </a-upload>
</template>

<script lang="ts" setup>
  import { message, Upload as AUpload } from 'ant-design-vue';
</script>
```

**上传函数**

```js
// 选中目录
const zipFile = ref<any>();
// 上传加载状态
const uploading = ref(false);
// 分片大小
const chunkSize = 100 * 1024 * 1024; // 100M
const beforeUploadDirectory = async (info) => {
  zipFile.value = info;
  if (zipFile.value) {
    if (!zipFile.value.name.endsWith('.zip')) {
      message.error('请选择zip文件!');
      return false;
    }
    uploading.value = true;
    try {
      // 总分片数
      const totalChunks = Math.ceil(zipFile.value.size / chunkSize);

      let res;
      // 遍历并逐个上传分片
      for (let chunkIndex = 0; chunkIndex < totalChunks; chunkIndex++) {
        const start = chunkIndex * chunkSize;
        const end = Math.min(zipFile.value.size, start + chunkSize);
        // 获取当前分片
        const chunk = zipFile.value.slice(start, end);
        // 封装FormData对象
        const formData = new FormData();
        formData.append('file', chunk);
        formData.append('chunkIndex', chunkIndex.toString());
        formData.append('totalChunks', totalChunks.toString());
        formData.append('fileName', zipFile.value.name);
        // 上传分片
        res = await importShootingPoint(formData, props.sideSlopeId);
        if (!res.isSuccess) {
          message.error(res.message);
          throw new Error('分片上传失败!');
        }
      }
      message.success('解析成功!');
    } catch (error) {
      message.error('解析失败,请重试!');
    } finally {
      uploading.value = false;
    }
  }
  return false;
};
```

### 后端部分

```c#
[HttpPost("ImportShootingPoint")]
[ProducesResponseType(typeof(BaseResult<IEnumerable<InspBasicShootingPointDto?>>), StatusCodes.Status200OK)]
[DisableRequestSizeLimit] // "Failed to read the request form. Request body too large. The max request body size is 30000000 bytes."
public async Task<IActionResult> ImportShootingPoint([FromForm] IFormFile file, [FromForm] int chunkIndex, [FromForm] int totalChunks, [FromForm] string fileName, [FromQuery] long id)
{
    try
    {
        // 临时文件存储目录
        var temFolder = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Uploads", id.ToString());
        if (!Directory.Exists(temFolder))
            Directory.CreateDirectory(temFolder);
        // 保存分片文件
        var chunkFilePath = Path.Combine(temFolder, $"{id}.part{chunkIndex}");
        using (var stream = new FileStream(chunkFilePath, FileMode.Create))
        {
            await file.CopyToAsync(stream);
        }
        // 检查是否是最后一片
        if (chunkIndex == totalChunks - 1)
        {
            // 合并所有分片文件
            var finalFilePath = Path.Combine(temFolder, id.ToString());
            using (var finalFileStream = new FileStream(finalFilePath, FileMode.Create))
            {
                for (int i = 0; i < totalChunks; i++)
                {
                    var partFilePath = Path.Combine(temFolder, $"{id}.part{i}");
                    using (var partStream = new FileStream(partFilePath, FileMode.Open))
                    {
                        await partStream.CopyToAsync(finalFileStream);
                    }
                    System.IO.File.Delete(partFilePath);
                }
            }
            // 合并完成，处理文件
            var fileInfo = new FileInfo(finalFilePath);
            using (var mergedFilStream = new FileStream(finalFilePath, FileMode.Open, FileAccess.Read))
            {
                var result = await _inspBasicSideSlopeService.UpdateInspBasicShootingPoints(mergedFilStream, id, fileName);
                return Ok(result);
            }
        }
        return Ok(new BaseResult(true));
    }
    catch (Exception ex)
    {
        // 处理错误
        return BadRequest(new { isSuccess = false, message = ex.Message });
    }
}
```

# 流&文件互转

> **流使用前记得使用`Seek(0, SeekOrigin.Begin)`重置指针位置**

### 内存流&文件流

```c#
// FileStream --> MemoryStream
using System.IO;

using (FileStream fileStream = new FileStream("filePath", FileMode.Open))
{
    using (MemoryStream memoryStream = new MemoryStream())
    {
        fileStream.CopyTo(memoryStream);
    }
}

```

```C#
// MemoryStream --> FileStream

memoryStream.Seek(0, SeekOrigin.Begin);
using (var fileStream = new FileStream("filepath", FileMode.Create, FileAccess.Write))
{
    await memoryStream.CopyToAsync(fileStream);
}
```

>`FileMode`
>
>- FileMode.**Append**
>  - 打开现有文件，并将数据添加到文件末尾；保证文件存在
>  - 与`FileAccess.Write`或`FileAccess.ReadWrite`一起使用
>
>- FileMode.**Create**：创建新文件，已存在则覆盖
>
>- FileMode.**CreateNew**：创建新文件，已存在则报错
>
>- FileMode.**Open**：打开现有文件，不存在则报错
>
>- FileMode.**OpenOrCreate**：打开一个文件，不存在则创建
>
>- FileMode.**Truncate**：打开并清空现有文件，不存在则报错
>
>`FileAccess`
>
>- FileAccess.**Read**：只允许读取文件内容
>
>- FileAccess.**Write**：只允许写入文件内容
>
>- FileAccess.**ReadWrite**：允许同时读取和写入文件

### 流&字节数组

```c#
// Stream --> byte[]
using System.IO;

using (MemoryStream memoryStream = new MemoryStream())
{
    byte[] data = memoryStream.ToArray();
}
```

```C#
// byte[] --> Stream

byte[] byteArray = new byte[] { /* 一些数据 */ };
MemoryStream memoryStream = new MemoryStream(byteArray);
```

### 文件&字节数组

```C#
// File --> byte[]

byte[] fileData = File.ReadAllBytes("filePath");
```

```c#
// byte[] --> File

byte[] data = new byte[] { /* 一些数据 */ };
File.WriteAllBytes("filePath", data);
```

