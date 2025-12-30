import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.util.ResourceUtils;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;
import java.util.UUID;

@Component
public class ResourceDirectoryWriter {
    
    @Value("${spring.profiles.active:default}")
    private String activeProfile;
    
    /**
     * 写入到 resources/output 目录（仅开发环境可用）
     */
    public String writeToResourcesOutput(List<String> lines) throws IOException {
        return writeToResourcesOutput(lines, "output");
    }
    
    public String writeToResourcesOutput(List<String> lines, String prefix) throws IOException {
        // 生成文件名
        String fileName = generateFileName(prefix);
        
        // 获取 resources 目录路径
        File resourcesDir = getResourcesDirectory();
        
        // 创建 output 子目录
        File outputDir = new File(resourcesDir, "output");
        if (!outputDir.exists()) {
            if (!outputDir.mkdirs()) {
                throw new IOException("无法创建目录: " + outputDir.getAbsolutePath());
            }
        }
        
        // 创建文件
        File outputFile = new File(outputDir, fileName);
        
        // 写入内容
        try (BufferedWriter writer = new BufferedWriter(
                new OutputStreamWriter(new FileOutputStream(outputFile), StandardCharsets.UTF_8))) {
            for (String line : lines) {
                writer.write(line);
                writer.newLine();
            }
        }
        
        return outputFile.getAbsolutePath();
    }
    
    /**
     * 获取 resources 目录
     */
    private File getResourcesDirectory() throws FileNotFoundException {
        try {
            // 方式1：通过 ResourceUtils
            File classpathRoot = ResourceUtils.getFile("classpath:");
            return classpathRoot;
        } catch (FileNotFoundException e) {
            // 方式2：通过 ClassLoader
            String classpath = getClass().getClassLoader().getResource("").getPath();
            return new File(classpath);
        }
    }
    
    /**
     * 生成文件名
     */
    private String generateFileName(String prefix) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss_SSS");
        String timestamp = LocalDateTime.now().format(formatter);
        String random = UUID.randomUUID().toString().substring(0, 8);
        
        if (prefix == null || prefix.trim().isEmpty()) {
            prefix = "output";
        }
        
        return String.format("%s_%s_%s.txt", prefix, timestamp, random);
    }
    
    /**
     * 检查是否在 JAR 中运行
     */
    public boolean isRunningInJar() {
        String protocol = getClass().getResource("").getProtocol();
        return "jar".equals(protocol);
    }
    
    /**
     * 安全写入（检测运行环境）
     */
    public String safeWriteToResources(List<String> lines) throws IOException {
        if (isRunningInJar()) {
            throw new IllegalStateException("不能在 JAR 包中写入 resources 目录");
        }
        
        if (!"dev".equals(activeProfile) && !"local".equals(activeProfile)) {
            throw new IllegalStateException("仅允许在开发环境写入 resources 目录");
        }
        
        return writeToResourcesOutput(lines);
    }
}
