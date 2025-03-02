---
title: An Audio Format Converter based on ffmpeg
---



# A small tool used for Audio Convertion
## Background
The .wav format is widely used in daily work, especially when mixing audio. However, when communicating with others and clients, the .mp3 format is often preferred.
Additionally, there are many other audio formats available today, such as .aac. Therefore, I need a tool to help me convert these audio files into .wav or .mp3 formats.
## Structure
### IsAudioFile?
if the files user drag in are not audio files,they are not allowed to effect.
```C#
// 处理拖拽事件
private void FileDragBox_DragEnter(object sender, DragEventArgs e)
{
 // 判断是否是文件类型的拖拽
 if (e.Data.GetDataPresent(DataFormats.FileDrop))
 {
  string[] files = (string[])e.Data.GetData(DataFormats.FileDrop);
  // 只接受音频文件（例如：wav, mp3）
  bool isAudioFile = files.All(file => IsAudioFile(file));
  e.Effect = isAudioFile ? DragDropEffects.Copy : DragDropEffects.None;
 }
 else
 {
  e.Effect = DragDropEffects.None;
 }
}
private void FileDragBox_DragDrop(object sender, DragEventArgs e)
{
    // 获取拖拽的文件路径
    string[] files = (string[])e.Data.GetData(DataFormats.FileDrop);
    if (files.Length > 0)
    {
        // 过滤出音频文件
        var audioFiles = files.Where(file => IsAudioFile(file)).ToArray();
        if (audioFiles.Length > 0)
        {
            // 显示拖拽的文件路径
            FileDragBox.Text = string.Join(Environment.NewLine, audioFiles);
        }
        else
        {
            MessageBox.Show("只支持音频文件！");
        }
    }
}

// 判断文件是否是音频文件（支持 .wav 和 .mp3 格式）
private bool IsAudioFile(string file)
 {
  string[] supportedExtensions = { ".wav", ".mp3", ".flac", ".aac", ".ogg" }; // 可以根据需要扩展
  return supportedExtensions.Contains(Path.GetExtension(file).ToLower());
 }
```
### GetIntoConvertion
```C#
 // 处理转换按钮点击事件
 private void convertButton_Click(object sender, EventArgs e)
 {
     string[] files = FileDragBox.Text.Split(new[] { Environment.NewLine }, StringSplitOptions.RemoveEmptyEntries);

     if (files.Length == 0)
     {
         MessageBox.Show("请先拖拽文件！");
         return;
     }

     // 获取用户选择的输出格式
     string selectedFormat = formatComboBox.SelectedItem.ToString().ToLower();
     string outputExtension = selectedFormat == "wav" ? ".wav" : ".mp3";

     // 执行批量转换
     foreach (var file in files)
     {
         string outputFilePath = Path.ChangeExtension(file, outputExtension);
         ConvertAudioFile(file, outputFilePath);
     }
 }
```
Accroding to the selection of  itemBox,generate a path that we need,which is the prepare for the convertion.

```C#
// 转换音频文件
public static void ConvertAudioFile(string inputFilePath, string outputFilePath)
{
    try
    {
        // 如果输入和输出格式相同，直接返回（不进行转换）
        if (Path.GetExtension(inputFilePath).ToLower() == Path.GetExtension(outputFilePath).ToLower())
        {
            MessageBox.Show("输入和输出文件格式相同，跳过转换。");
            return;
        }

        // 检查目标文件是否存在
        if (File.Exists(outputFilePath))
        {
            // 提示用户是否覆盖文件
            var result = MessageBox.Show($"文件 {outputFilePath} 已存在，是否覆盖?", "确认覆盖", MessageBoxButtons.YesNo, MessageBoxIcon.Warning);
            if (result == DialogResult.Yes)
            {
                // 删除目标文件
                File.Delete(outputFilePath);
            }
            else
            {
                return; // 用户选择不覆盖，跳过转换
            }
        }

        // 确保 FFmpeg 可用
        string ffmpegPath = "ffmpeg"; // 假设 FFmpeg 已经在系统环境变量中

        // 设置命令行参数
        string arguments = $"-i \"{inputFilePath}\" \"{outputFilePath}\"";

        // 启动进程执行 FFmpeg 命令
        ProcessStartInfo startInfo = new ProcessStartInfo
        {
            FileName = ffmpegPath,
            Arguments = arguments,
            CreateNoWindow = true,  // 不显示命令行窗口
            UseShellExecute = false,
            RedirectStandardOutput = true, // 重定向标准输出
            RedirectStandardError = true   // 重定向标准错误
        };

        Process process = Process.Start(startInfo);

        // 设置进程超时
        bool exited = process.WaitForExit(30000);  // 30秒超时
        if (!exited)
        {
            process.Kill();
            MessageBox.Show("转换超时！");
            return;
        }

        // 获取 FFmpeg 的标准输出和错误信息
        string output = process.StandardOutput.ReadToEnd();
        string error = process.StandardError.ReadToEnd();

        // 输出 FFmpeg 的日志信息（可用于调试）
        Console.WriteLine("FFmpeg Output: " + output);
        Console.WriteLine("FFmpeg Error: " + error);

        // 检查 FFmpeg 进程的退出码，0 表示成功
        if (process.ExitCode != 0)
        {
            // 只有当有错误信息时才弹出错误
            if (!string.IsNullOrEmpty(error))
            {
                MessageBox.Show($"转换失败: {error}", "错误", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
            else
            {
                // 如果没有错误输出，但退出码非零，可以显示一个通用错误提示
                MessageBox.Show("转换失败，但没有提供错误信息。", "错误", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
        else
        {
            // 如果退出码为 0，表示转换成功
            MessageBox.Show($"转换完成: {outputFilePath}", "成功", MessageBoxButtons.OK, MessageBoxIcon.Information);
        }

    }
    catch (Exception ex)
    {
        MessageBox.Show($"错误: {ex.Message}");
    }
}
```
Based on the selection of the itemBox, generate the necessary paths, which will prepare for the conversion. Using the input and output paths, FFmpeg will automatically generate the desired audio file by starting an external process.