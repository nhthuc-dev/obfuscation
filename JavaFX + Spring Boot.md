# Hướng dẫn xây dựng ứng dụng JavaFX + Spring Boot: Mã hóa với ProGuard, đóng gói với jlink, jpackage, và tham khảo Gluon & GraalVM Native Image

Hướng dẫn này cung cấp các bước chi tiết để xây dựng một ứng dụng JavaFX tích hợp Spring Boot, mã hóa mã nguồn bằng ProGuard để bảo vệ mã, và đóng gói ứng dụng thành file thực thi độc lập bằng jlink và jpackage. Phần bổ sung sẽ đề cập đến việc sử dụng Gluon và GraalVM Native Image trên Windows để tạo ứng dụng native.

## Yêu cầu

- **JDK**: 17 hoặc cao hơn (đề xuất OpenJDK hoặc AdoptOpenJDK).
- **Maven** hoặc **Gradle**: Cài đặt công cụ build phù hợp.
- **JavaFX**: Phiên bản tương thích với JDK (ví dụ: JavaFX 17).
- **ProGuard**: Phiên bản mới nhất (ví dụ: 7.3.2).
- **jpackage**: Có sẵn trong JDK 14+.
- **Spring Boot**: Phiên bản 3.x (tương thích với Java 17).
- **Gluon (tùy chọn)**: Gluon Client plugin để hỗ trợ JavaFX trên các nền tảng.
- **GraalVM (tùy chọn)**: GraalVM Community Edition 22.3+ (hoặc mới hơn) để tạo native image.
- **Hệ điều hành**: Windows, macOS, hoặc Linux (phần GraalVM tập trung vào Windows).

## Cấu trúc dự án mẫu

Dưới đây là cấu trúc thư mục mẫu:

```
my-javafx-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   ├── com.example.myapp/
│   │   │   │   ├── Application.java
│   │   │   │   ├── MainApp.java
│   │   │   │   ├── controller/
│   │   │   │   ├── service/
│   │   │   │   ├── fxml/
│   │   │   │   │   └── main.fxml
│   │   ├── resources/
│   │   │   ├── application.properties
│   │   │   ├── fxml/
│   │   │   │   └── main.fxml
├── proguard-rules.pro
├── pom.xml (hoặc build.gradle)
├── README.md
```

## Bước 1: Tạo ứng dụng JavaFX + Spring Boot

### File `Application.java` (Spring Boot Entry)

```java
package com.example.myapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        MainApp.main(args); // Gọi JavaFX
    }
}
```

### File `MainApp.java` (JavaFX Entry)

```java
package com.example.myapp;

import javafx.application.Application;
import javafx.fxml.FXMLLoader;
import javafx.scene.Scene;
import javafx.stage.Stage;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MainApp extends Application {
    private ApplicationContext context;

    @Override
    public void init() {
        context = new AnnotationConfigApplicationContext(Application.class);
    }

    @Override
    public void start(Stage primaryStage) throws Exception {
        FXMLLoader loader = new FXMLLoader(getClass().getResource("/fxml/main.fxml"));
        loader.setControllerFactory(context::getBean);
        Scene scene = new Scene(loader.load(), 800, 600);
        primaryStage.setScene(scene);
        primaryStage.setTitle("My JavaFX App");
        primaryStage.show();
    }
}
```

### File `application.properties`

```properties
spring.application.name=my-javafx-app
```

### File `main.fxml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?import javafx.scene.layout.VBox?>
<VBox xmlns="http://javafx.com/javafx" xmlns:fx="http://javafx.com/fxml">
    <children>
        <fx:include fx:id="content" source="content.fxml"/>
    </children>
</VBox>
```

## Bước 2: Cấu hình ProGuard để mã hóa

### File `proguard-rules.pro`

```proguard
# Giữ các lớp JavaFX và Spring Boot cần thiết
-keep class com.example.myapp.** { *; }
-keep class org.springframework.** { *; }
-keep class javafx.** { *; }

# Giữ các annotation và cấu hình Spring
-keepattributes *Annotation*,Signature,EnclosingMethod
-keepclassmembers,allowobfuscation class * {
    @org.springframework.beans.factory.annotation.Autowired *;
    @org.springframework.stereotype.Component *;
}

# Giữ các lớp FXML và Controller
-keepclassmembers class **Controller {
    public *;
}

# Tắt tối ưu hóa không cần thiết
-dontoptimize
-dontshrink

# Giữ các thuộc tính và phương thức cần thiết cho JavaFX
-keepclassmembers class * extends javafx.application.Application {
    public *;
}

# Giữ các thuộc tính getter/setter
-keepclassmembers class * {
    public void set*(***);
    public *** get*();
}

# Bỏ qua cảnh báo
-dontwarn
```

## Bước 3: Cấu hình build với Maven

### File `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-javafx-app</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>17</java.version>
        <javafx.version>17.0.2</javafx.version>
        <spring-boot.version>3.2.0</spring-boot.version>
        <proguard.version>7.3.2</proguard.version>
    </properties>

    <dependencies>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>${spring-boot.version}</version>
        </dependency>

        <!-- JavaFX -->
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-controls</artifactId>
            <version>${javafx.version}</version>
        </dependency>
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-fxml</artifactId>
            <version>${javafx.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Spring Boot Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
            </plugin>

            <!-- Maven Compiler Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>

            <!-- ProGuard Plugin -->
            <plugin>
                <groupId>com.github.wvengen</groupId>
                <artifactId>proguard-maven-plugin</artifactId>
                <version>2.0.18</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>proguard</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <proguardInclude>${project.basedir}/proguard-rules.pro</proguardInclude>
                    <options>
                        <option>-outjars ${project.build.directory}/obfuscated.jar</option>
                    </options>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>net.sf.proguard</groupId>
                        <artifactId>proguard-base</artifactId>
                        <version>${proguard.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>
```

### Lệnh Maven

1. **Build và mã hóa với ProGuard**:

   ```bash
   mvn clean package
   ```

   File JAR đã mã hóa sẽ nằm tại: `target/obfuscated.jar`.

2. **Tạo runtime image với jlink**:

   ```bash
   jlink --module-path "path/to/javafx-jmods-${javafx.version};target/obfuscated.jar" \
         --add-modules com.example.myapp,javafx.controls,javafx.fxml \
         --output target/jre
   ```

3. **Đóng gói ứng dụng với jpackage**:

   ```bash
   jpackage --type app-image \
            --input target \
            --name MyJavaFXApp \
            --main-jar obfuscated.jar \
            --main-class com.example.myapp.Application \
            --java-options "--module-path target/jre --add-modules com.example.myapp,javafx.controls,javafx.fxml" \
            --dest target
   ```

## Bước 4: Cấu hình build với Gradle

### File `build.gradle`

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.0'
    id 'org.beryx.jlink' version '2.25.0'
    id 'com.github.johnrengelman.shadow' version '7.1.2'
    id 'proguard' version '7.3.2'
}

group 'com.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter:3.2.0'
    implementation 'org.openjfx:javafx-controls:17.0.2'
    implementation 'org.openjfx:javafx-fxml:17.0.2'
}

proguard {
    configuration 'proguard-rules.pro'
    injars 'build/libs/my-javafx-app-${version}.jar'
    outjars 'build/libs/obfuscated.jar'
}

jlink {
    launcher {
        name = 'MyJavaFXApp'
        mainClass = 'com.example.myapp.Application'
    }
    jpackage {
        outputDir = 'build/jpackage'
        imageOptions = ['--java-options', '--module-path build/jre --add-modules com.example.myapp,javafx.controls,javafx.fxml']
    }
}

tasks.jlink.dependsOn tasks.proguard
tasks.jpackage.dependsOn tasks.jlink
```

### Lệnh Gradle

1. **Build và mã hóa với ProGuard**:

   ```bash
   gradle clean build proguard
   ```

   File JAR đã mã hóa sẽ nằm tại: `build/libs/obfuscated.jar`.

2. **Tạo runtime image với jlink**:

   ```bash
   gradle jlink
   ```

   Runtime image sẽ nằm tại: `build/image`.

3. **Đóng gói ứng dụng với jpackage**:

   ```bash
   gradle jpackage
   ```

   Ứng dụng đóng gói sẽ nằm tại: `build/jpackage`.

## Bước 5: Tham khảo Gluon và GraalVM Native Image (Windows)

### Sử dụng Gluon

Gluon cung cấp các công cụ để xây dựng ứng dụng JavaFX đa nền tảng, đặc biệt là với plugin **Gluon Client** để tạo native image hoặc đóng gói ứng dụng cho desktop và mobile.

#### Cấu hình Gluon với Maven

1. Thêm Gluon Client plugin vào `pom.xml`:

   ```xml
   <plugin>
       <groupId>com.gluonhq</groupId>
       <artifactId>client-maven-plugin</artifactId>
       <version>0.1.38</version>
       <configuration>
           <target>host</target>
           <mainClass>com.example.myapp.MainApp</mainClass>
       </configuration>
   </plugin>
   ```

2. Thêm dependency:

   ```xml
   <dependency>
       <groupId>com.gluonhq</groupId>
       <artifactId>client</artifactId>
       <version>0.1.38</version>
   </dependency>
   ```

3. **Lệnh Gluon**:

   ```bash
   mvn client:build client:run
   ```

   Lệnh này sẽ biên dịch và chạy ứng dụng JavaFX trên nền tảng Windows.

4. **Đóng gói native image với Gluon**:

   ```bash
   mvn client:package
   ```

   Kết quả sẽ tạo ra một ứng dụng native trong thư mục `target/client`.

#### Lưu ý Gluon

- Gluon Client yêu cầu GraalVM để tạo native image.
- Đảm bảo cấu hình đúng `JAVA_HOME` trỏ đến GraalVM.

### Sử dụng GraalVM Native Image

GraalVM Native Image cho phép biên dịch ứng dụng JavaFX + Spring Boot thành file thực thi native, giảm thời gian khởi động và kích thước ứng dụng.

#### Yêu cầu (Windows)

- **GraalVM**: Tải và cài đặt GraalVM Community Edition (22.3+).
- **Microsoft Visual Studio**: Cài đặt Visual Studio 2022 (Community Edition) với thành phần "Desktop development with C++".
- **Windows SDK**: Cần thiết cho native image.

#### Cài đặt GraalVM Native Image

1. Cài đặt GraalVM và thiết lập `JAVA_HOME`:

   ```bash
   set JAVA_HOME=C:\path\to\graalvm
   set PATH=%JAVA_HOME%\bin;%PATH%
   ```

2. Cài đặt `native-image`:

   ```bash
   gu install native-image
   ```

#### Cấu hình Maven cho GraalVM

1. Thêm plugin `native-maven-plugin` vào `pom.xml`:

   ```xml
   <plugin>
       <groupId>org.graalvm.buildtools</groupId>
       <artifactId>native-maven-plugin</artifactId>
       <version>0.9.20</version>
       <executions>
           <execution>
               <goals>
                   <goal>compile-no-fork</goal>
               </goals>
           </execution>
       </executions>
       <configuration>
           <mainClass>com.example.myapp.Application</mainClass>
           <imageName>my-javafx-app</imageName>
           <buildArgs>
               <arg>--no-fallback</arg>
               <arg>--enable-preview</arg>
               <arg>-H:+ReportExceptionStackTraces</arg>
           </buildArgs>
       </configuration>
   </plugin>
   ```

2. Thêm file cấu hình GraalVM (`src/main/resources/META-INF/native-image/reflect-config.json`):

   ```json
   [
       {
           "name": "com.example.myapp.Application",
           "allDeclaredConstructors": true,
           "allPublicMethods": true
       },
       {
           "name": "com.example.myapp.MainApp",
           "allDeclaredConstructors": true,
           "allPublicMethods": true
       }
   ]
   ```

3. **Biên dịch Native Image**:

   ```bash
   mvn -Pnative package
   ```

   File thực thi sẽ nằm tại: `target/my-javafx-app.exe`.

#### Cấu hình Gradle cho GraalVM

1. Thêm plugin `org.graalvm.buildtools.native` vào `build.gradle`:

   ```gradle
   plugins {
       id 'org.graalvm.buildtools.native' version '0.9.20'
   }

   graalvmNative {
       binaries {
           main {
               mainClass = 'com.example.myapp.Application'
               imageName = 'my-javafx-app'
               buildArgs.addAll(['--no-fallback', '--enable-preview', '-H:+ReportExceptionStackTraces'])
           }
       }
   }
   ```

2. **Biên dịch Native Image**:

   ```bash
   gradle nativeCompile
   ```

   File thực thi sẽ nằm tại: `build/native/nativeCompile/my-javafx-app.exe`.

#### Lưu ý GraalVM

- GraalVM Native Image yêu cầu cấu hình reflection (`reflect-config.json`) cho Spring Boot và JavaFX.
- Kích thước file thực thi nhỏ hơn và khởi động nhanh hơn so với jpackage.
- Đảm bảo cài đặt Visual Studio và Windows SDK để tránh lỗi biên dịch trên Windows.

## Bước 6: Chạy ứng dụng

- **Chạy từ JAR**:

  ```bash
  java -jar target/obfuscated.jar
  ```

- **Chạy từ jpackage**:

  - Windows: `target/MyJavaFXApp/MyJavaFXApp.exe`
  - macOS/Linux: `build/jpackage/MyJavaFXApp/MyJavaFXApp`

- **Chạy từ Native Image (Windows)**:

  ```bash
  target/my-javafx-app.exe
  ```

## Lưu ý

- Đảm bảo các module JavaFX (`javafx.controls`, `javafx.fxml`) được thêm vào `--add-modules` trong jlink/jpackage.
- File `proguard-rules.pro` cần được điều chỉnh nếu có thêm thư viện hoặc cấu hình đặc biệt.
- Gluon và GraalVM yêu cầu cấu hình bổ sung cho Spring Boot (reflection, resources).
- Kiểm tra tính tương thích của JavaFX và JDK trên hệ điều hành của bạn.
- GraalVM trên Windows yêu cầu Visual Studio và Windows SDK được cài đặt đúng.
