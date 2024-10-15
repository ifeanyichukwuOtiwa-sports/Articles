To write a Parquet file using Apache Hadoop, push it to an S3 bucket, and then zip the file, you can follow these steps:

1. **Set Up Your Hadoop Environment:**
   - Ensure you have Hadoop installed and configured.
   - Make sure you have the required libraries for handling Parquet files and AWS S3.

2. **Write Data to a Parquet File:**

Here's a sample code snippet in Java to write data to a Parquet file using Apache Hadoop:

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hdfs.DistributedFileSystem;
import org.apache.parquet.hadoop.ParquetWriter;
import org.apache.parquet.hadoop.example.ExampleParquetWriter;
import org.apache.parquet.example.data.simple.SimpleGroup;
import org.apache.parquet.example.data.simple.NanoTime;
import org.apache.parquet.example.data.simple.Timestamp;
import org.apache.parquet.schema.MessageType;
import org.apache.parquet.schema.MessageTypeParser;

public class ParquetWriteExample {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "hdfs://localhost:9000");
        conf.set("fs.hdfs.impl", DistributedFileSystem.class.getName());
        
        String schemaString = "message schema { "
                + "required binary name (UTF8); "
                + "required int32 age; "
                + "required boolean isStudent; "
                + "}";
        MessageType schema = MessageTypeParser.parseMessageType(schemaString);

        Path path = new Path("/user/hadoop/output.parquet");
        ParquetWriter<SimpleGroup> writer = ExampleParquetWriter.builder(path)
                .withConf(conf)
                .withType(schema)
                .build();

        SimpleGroup group = new SimpleGroup(schema);
        group.add("name", "John Doe");
        group.add("age", 25);
        group.add("isStudent", false);
        writer.write(group);
        
        writer.close();
    }
}
```

3. **Push the Parquet File to S3:**

You can use the AWS SDK for Java to upload the Parquet file to an S3 bucket. Ensure you have AWS credentials configured.

```java
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import com.amazonaws.services.s3.model.PutObjectRequest;

import java.io.File;

public class S3UploadExample {
    public static void main(String[] args) {
        String accessKey = "your-access-key";
        String secretKey = "your-secret-key";
        String bucketName = "your-bucket-name";
        String keyName = "output.parquet";

        BasicAWSCredentials awsCreds = new BasicAWSCredentials(accessKey, secretKey);
        AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
                .withRegion("your-region")
                .withCredentials(new AWSStaticCredentialsProvider(awsCreds))
                .build();

        File file = new File("/path/to/your/output.parquet");
        s3Client.putObject(new PutObjectRequest(bucketName, keyName, file));
    }
}
```

4. **Zip the File:**

To zip the Parquet file before uploading it to S3, you can use Java's built-in `java.util.zip` package.

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

public class ZipFileExample {
    public static void main(String[] args) {
        String sourceFile = "/path/to/your/output.parquet";
        String zipFile = "/path/to/your/output.zip";

        try {
            FileOutputStream fos = new FileOutputStream(zipFile);
            ZipOutputStream zos = new ZipOutputStream(fos);
            ZipEntry zipEntry = new ZipEntry("output.parquet");
            zos.putNextEntry(zipEntry);

            FileInputStream fis = new FileInputStream(sourceFile);
            byte[] buffer = new byte[1024];
            int length;
            while ((length = fis.read(buffer)) >= 0) {
                zos.write(buffer, 0, length);
            }
            zos.closeEntry();
            fis.close();
            zos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

You can then upload the zipped file to S3 using the same AWS SDK method shown above, adjusting the file path and key name accordingly.

By combining these steps, you can write data to a Parquet file using Apache Hadoop, zip the file, and upload it to an S3 bucket.



Sure! Here is an updated example using Java 21 and a proper Parquet writer without relying on the ExampleParquetWriter. 

### Steps:

1. **Set Up Your Environment:**
   - Ensure you have Java 21 installed.
   - Ensure you have Hadoop, Parquet libraries, and AWS SDK for Java included in your project dependencies (e.g., via Maven or Gradle).

2. **Write Data to a Parquet File Using Apache Parquet:**

Here’s an example using the Parquet API to write data to a Parquet file:

#### Maven Dependencies:
Make sure to add the necessary dependencies to your `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>3.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.parquet</groupId>
        <artifactId>parquet-hadoop</artifactId>
        <version>1.12.3</version>
    </dependency>
    <dependency>
        <groupId>com.amazonaws</groupId>
        <artifactId>aws-java-sdk-s3</artifactId>
        <version>1.12.341</version>
    </dependency>
</dependencies>
```

#### Java Code:
Here's the Java code to write to a Parquet file, zip it, and upload it to S3:

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.parquet.hadoop.ParquetWriter;
import org.apache.parquet.hadoop.ParquetFileWriter;
import org.apache.parquet.hadoop.metadata.CompressionCodecName;
import org.apache.parquet.hadoop.metadata.CompressionCodecName;
import org.apache.parquet.schema.MessageType;
import org.apache.parquet.schema.MessageTypeParser;
import org.apache.parquet.hadoop.example.GroupWriteSupport;
import org.apache.parquet.example.data.simple.SimpleGroupFactory;
import org.apache.parquet.example.data.simple.SimpleGroup;
import org.apache.parquet.hadoop.api.WriteSupport;
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import com.amazonaws.services.s3.model.PutObjectRequest;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.StandardCopyOption;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

public class ParquetWriteExample {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "file:///");
        
        String schemaString = "message schema { "
                + "required binary name (UTF8); "
                + "required int32 age; "
                + "required boolean isStudent; "
                + "}";
        MessageType schema = MessageTypeParser.parseMessageType(schemaString);
        GroupWriteSupport.setSchema(schema, conf);

        Path path = new Path("output.parquet");
        ParquetWriter<SimpleGroup> writer = new ParquetWriter<>(
                path,
                new GroupWriteSupport(),
                CompressionCodecName.SNAPPY,
                ParquetWriter.DEFAULT_BLOCK_SIZE,
                ParquetWriter.DEFAULT_PAGE_SIZE,
                true,
                conf
        );

        SimpleGroupFactory factory = new SimpleGroupFactory(schema);
        SimpleGroup group = factory.newGroup()
                .append("name", "John Doe")
                .append("age", 25)
                .append("isStudent", false);
        writer.write(group);
        writer.close();

        // Zip the Parquet file
        String sourceFile = "output.parquet";
        String zipFile = "output.zip";
        zipFile(sourceFile, zipFile);

        // Upload the zipped file to S3
        uploadToS3(zipFile, "your-access-key", "your-secret-key", "your-bucket-name", "output.zip");
    }

    private static void zipFile(String sourceFile, String zipFile) throws IOException {
        try (FileOutputStream fos = new FileOutputStream(zipFile);
             ZipOutputStream zos = new ZipOutputStream(fos);
             FileInputStream fis = new FileInputStream(sourceFile)) {

            ZipEntry zipEntry = new ZipEntry(new File(sourceFile).getName());
            zos.putNextEntry(zipEntry);

            byte[] buffer = new byte[1024];
            int length;
            while ((length = fis.read(buffer)) >= 0) {
                zos.write(buffer, 0, length);
            }

            zos.closeEntry();
        }
    }

    private static void uploadToS3(String filePath, String accessKey, String secretKey, String bucketName, String keyName) {
        BasicAWSCredentials awsCreds = new BasicAWSCredentials(accessKey, secretKey);
        AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
                .withRegion("your-region")
                .withCredentials(new AWSStaticCredentialsProvider(awsCreds))
                .build();

        File file = new File(filePath);
        s3Client.putObject(new PutObjectRequest(bucketName, keyName, file));
    }
}
```

This example demonstrates how to write data to a Parquet file using Apache Parquet, compress the file using Snappy, zip it, and upload it to an S3 bucket. Adjust the `accessKey`, `secretKey`, `bucketName`, and `region` with your AWS credentials and configurations.



The logical flow for converting records from a database to Parquet files and storing them in S3 buckets involves several key steps. Here’s a detailed outline:

### Logical Flow

1. **Extract Data from Database:**
   - Connect to the database.
   - Execute queries to retrieve the necessary records.
   - Fetch the data into your application memory.

2. **Transform Data (if necessary):**
   - Perform any required data transformations, such as data cleaning, type conversions, or aggregations.
   - Map the data to the Parquet schema.

3. **Write Data to Parquet Files:**
   - Define the Parquet schema.
   - Create Parquet writer instances.
   - Write the data into Parquet files.

4. **Compress and Archive the Files (optional):**
   - Compress the Parquet files using a suitable compression algorithm (e.g., Snappy).
   - Optionally, archive the files into a ZIP format.

5. **Upload Parquet Files to S3:**
   - Connect to AWS S3 using the AWS SDK.
   - Upload the Parquet files to the desired S3 bucket and key (path).

6. **Clean Up (optional):**
   - Delete temporary files.
   - Log the completion status.

### Detailed Steps with Example Code

#### 1. Extract Data from Database

Use JDBC to connect to the database and retrieve data.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class DBExtractor {
    public ResultSet extractData(String query) throws Exception {
        String url = "jdbc:mysql://localhost:3306/your_database";
        String user = "your_username";
        String password = "your_password";

        Connection conn = DriverManager.getConnection(url, user, password);
        Statement stmt = conn.createStatement();
        return stmt.executeQuery(query);
    }
}
```

#### 2. Transform Data (if necessary)

Transform the ResultSet data into a suitable format for Parquet.

#### 3. Write Data to Parquet Files

Use Apache Parquet to write data into Parquet files.

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.parquet.hadoop.ParquetWriter;
import org.apache.parquet.hadoop.example.GroupWriteSupport;
import org.apache.parquet.schema.MessageType;
import org.apache.parquet.schema.MessageTypeParser;
import org.apache.parquet.example.data.simple.SimpleGroupFactory;
import org.apache.parquet.example.data.simple.SimpleGroup;

import java.sql.ResultSet;

public class ParquetWriterExample {
    public void writeParquet(ResultSet resultSet) throws Exception {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "file:///");

        String schemaString = "message schema { "
                + "required binary name (UTF8); "
                + "required int32 age; "
                + "required boolean isStudent; "
                + "}";
        MessageType schema = MessageTypeParser.parseMessageType(schemaString);
        GroupWriteSupport.setSchema(schema, conf);

        Path path = new Path("output.parquet");
        ParquetWriter<SimpleGroup> writer = new ParquetWriter<>(
                path,
                new GroupWriteSupport(),
                ParquetWriter.DEFAULT_COMPRESSION_CODEC_NAME,
                ParquetWriter.DEFAULT_BLOCK_SIZE,
                ParquetWriter.DEFAULT_PAGE_SIZE,
                true,
                conf
        );

        SimpleGroupFactory factory = new SimpleGroupFactory(schema);

        while (resultSet.next()) {
            SimpleGroup group = factory.newGroup()
                    .append("name", resultSet.getString("name"))
                    .append("age", resultSet.getInt("age"))
                    .append("isStudent", resultSet.getBoolean("isStudent"));
            writer.write(group);
        }

        writer.close();
    }
}
```

#### 4. Compress and Archive the Files (optional)

Compress and archive the Parquet file if needed.

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

public class FileCompressor {
    public void zipFile(String sourceFile, String zipFile) throws IOException {
        try (FileOutputStream fos = new FileOutputStream(zipFile);
             ZipOutputStream zos = new ZipOutputStream(fos);
             FileInputStream fis = new FileInputStream(sourceFile)) {

            ZipEntry zipEntry = new ZipEntry(new File(sourceFile).getName());
            zos.putNextEntry(zipEntry);

            byte[] buffer = new byte[1024];
            int length;
            while ((length = fis.read(buffer)) >= 0) {
                zos.write(buffer, 0, length);
            }

            zos.closeEntry();
        }
    }
}
```

#### 5. Upload Parquet Files to S3

Upload the Parquet file to S3 using the AWS SDK.

```java
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import com.amazonaws.services.s3.model.PutObjectRequest;

import java.io.File;

public class S3Uploader {
    public void uploadToS3(String filePath, String accessKey, String secretKey, String bucketName, String keyName) {
        BasicAWSCredentials awsCreds = new BasicAWSCredentials(accessKey, secretKey);
        AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
                .withRegion("your-region")
                .withCredentials(new AWSStaticCredentialsProvider(awsCreds))
                .build();

        File file = new File(filePath);
        s3Client.putObject(new PutObjectRequest(bucketName, keyName, file));
    }
}
```

#### 6. Clean Up (optional)

Delete temporary files and log the completion status.

### Main Execution Flow

Combine the above steps in a main execution class.

```java
public class Main {
    public static void main(String[] args) {
        try {
            DBExtractor extractor = new DBExtractor();
            ResultSet resultSet = extractor.extractData("SELECT name, age, isStudent FROM your_table");

            ParquetWriterExample writer = new ParquetWriterExample();
            writer.writeParquet(resultSet);

            FileCompressor compressor = new FileCompressor();
            compressor.zipFile("output.parquet", "output.zip");

            S3Uploader uploader = new S3Uploader();
            uploader.uploadToS3("output.zip", "your-access-key", "your-secret-key", "your-bucket-name", "output.zip");

            System.out.println("Process completed successfully.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

This complete flow ensures data is extracted from the database, written to Parquet files, optionally compressed and archived, and uploaded to an S3 bucket.