⚙️ Xây dựng ứng dụng JavaFX + Spring Boot: Làm rối mã với ProGuard, đóng gói với jlink & jpackage
Tài liệu này phác thảo quy trình xây dựng một ứng dụng desktop JavaFX mạnh mẽ được tích hợp với Spring Boot, tập trung vào các khía cạnh triển khai quan trọng: làm rối mã nguồn bằng ProGuard, tạo hình ảnh thời gian chạy tùy chỉnh với jlink, và tạo trình cài đặt dành riêng cho nền tảng với jpackage.

1. Giới thiệu
Việc kết hợp JavaFX cho giao diện người dùng phía client phong phú và Spring Boot cho các dịch vụ backend (ngay cả khi được nhúng trong cùng một ứng dụng) mang lại trải nghiệm phát triển mạnh mẽ. Tuy nhiên, việc triển khai một ứng dụng như vậy đòi hỏi phải xem xét cẩn thận các vấn đề bảo mật (làm rối mã) và phân phối (đóng gói). Hướng dẫn này sẽ đưa bạn qua các bước này.

2. Điều kiện tiên quyết
Trước khi bắt đầu, hãy đảm bảo bạn đã cài đặt những điều sau:

Java Development Kit (JDK) 11 trở lên: jlink và jpackage là một phần của JDK.

Maven hoặc Gradle: Để quản lý dự án và tự động hóa quá trình xây dựng.

ProGuard: Mặc dù thường được tích hợp thông qua các plugin xây dựng, việc hiểu cách sử dụng độc lập của nó cũng rất hữu ích.

3. Thiết lập dự án (Tổng quan ngắn gọn)
Thiết lập ứng dụng JavaFX + Spring Boot điển hình bao gồm:

Dự án Maven/Gradle: Cấu trúc dự án tiêu chuẩn.

pom.xml (Maven) hoặc build.gradle (Gradle): Các phụ thuộc cho JavaFX, Spring Boot, và có thể các plugin xây dựng cho ProGuard, jlink, và jpackage.

Lớp ứng dụng chính: Một lớp mở rộng javafx.application.Application và được chú thích bằng @SpringBootApplication. Phương thức start() của ứng dụng JavaFX thường sẽ khởi tạo ngữ cảnh Spring.

Ví dụ cấu trúc lớp Application:

import javafx.application.Application;
import javafx.stage.Stage;
import org.springframework.boot.SpringApplication;
import org.springframework.context.ConfigurableApplicationContext;

public class YourJavaFxApplication extends Application {

    private ConfigurableApplicationContext springContext;

    @Override
    public void init() throws Exception {
        // Khởi tạo ngữ cảnh Spring
        springContext = SpringApplication.run(YourJavaFxApplication.class);
    }

    @Override
    public void start(Stage primaryStage) throws Exception {
        // Tải FXML, lấy các bộ điều khiển từ ngữ cảnh Spring, v.v.
        // Ví dụ: FXMLLoader fxmlLoader = new FXMLLoader(getClass().getResource("/path/to/your/view.fxml"));
        // fxmlLoader.setControllerFactory(springContext::getBean); // Tích hợp Spring với FXML
        // Parent root = fxmlLoader.load();
        // primaryStage.setScene(new Scene(root));
        // primaryStage.setTitle("Ứng dụng của bạn");
        // primaryStage.show();
    }

    @Override
    public void stop() throws Exception {
        // Đóng ngữ cảnh Spring khi ứng dụng JavaFX dừng
        springContext.close();
    }

    public static void main(String[] args) {
        launch(args);
    }
}

4. Làm rối mã ProGuard
ProGuard là một công cụ miễn phí để thu nhỏ, tối ưu hóa, làm rối mã và tiền kiểm tra các tệp lớp Java. Nó phát hiện và loại bỏ các lớp, trường, phương thức và thuộc tính không sử dụng, tối ưu hóa mã bytecode và đổi tên các lớp, trường và phương thức còn lại bằng các tên ngắn, vô nghĩa.

Tại sao phải làm rối mã?
Bảo mật: Làm cho việc đảo ngược kỹ thuật trở nên khó khăn hơn, bảo vệ sở hữu trí tuệ.

Giảm kích thước: Loại bỏ mã không sử dụng, dẫn đến các tệp JAR nhỏ hơn.

Hiệu suất: Tối ưu hóa mã bytecode, có khả năng cải thiện hiệu suất thời gian chạy.

Cấu hình ProGuard (proguard.pro)
Việc tích hợp ProGuard thường liên quan đến một plugin Maven (ví dụ: proguard-maven-plugin) hoặc plugin Gradle. Phần cốt lõi của nó là tệp cấu hình.

Đây là một ví dụ proguard.pro cơ bản cho ứng dụng JavaFX + Spring Boot. Đây là điểm khởi đầu và có thể sẽ cần điều chỉnh dựa trên phản xạ, thư viện bên ngoài và cách sử dụng FXML cụ thể của ứng dụng của bạn.

# Cài đặt ProGuard chung
-keepattributes Signature,Exceptions,InnerClasses,SourceFile,LineNumberTable,*Annotation*
-dontskipnonpubliclibraryclasses
-dontskipnonpubliclibraryclassmembers

# Ngăn chặn cảnh báo cho các vấn đề phổ biến, điều chỉnh khi cần
-dontwarn java.lang.invoke.**
-dontwarn org.springframework.boot.autoconfigure.**
-dontwarn org.apache.tomcat.**
-dontwarn javafx.**

# Giữ lại lớp ứng dụng JavaFX chính
-keep public class com.yourcompany.yourapp.YourJavaFxApplication {
    public static void main(java.lang.String[]);
}

# Giữ lại các bộ điều khiển và phương thức FXML của JavaFX
-keep public class * extends javafx.application.Application {
    public <fields>;
    public <methods>;
}
-keep public class * implements javafx.fxml.Initializable {
    public <fields>;
    public <methods>;
}
-keep public class * {
    @javafx.fxml.FXML <fields>;
    @javafx.fxml.FXML <methods>;
}

# Giữ lại các điểm vào và cấu hình ứng dụng Spring Boot
-keep @org.springframework.boot.autoconfigure.SpringBootApplication class * {
    public static void main(java.lang.String[]);
}
-keep @org.springframework.context.annotation.Configuration class * {
    public <methods>;
}
-keep @org.springframework.stereotype.Component class * {
    public <fields>;
    public <methods>;
}
-keep @org.springframework.stereotype.Service class * {
    public <fields>;
    public <methods>;
}
-keep @org.springframework.stereotype.Repository class * {
    public <fields>;
    public <methods>;
}
-keep @org.springframework.web.bind.annotation.RestController class * {
    public <fields>;
    public <methods>;
}
-keep @org.springframework.web.bind.annotation.Controller class * {
    public <fields>;
    public <methods>;
}

# Giữ lại các lớp sử dụng phản xạ (ví dụ: @Autowired, @Value)
-keepclassmembers class * {
    @org.springframework.beans.factory.annotation.Autowired <fields>;
    @org.springframework.beans.factory.annotation.Autowired <methods>;
    @org.springframework.beans.factory.annotation.Value <fields>;
    @org.springframework.beans.factory.annotation.Value <methods>;
    @javax.annotation.PostConstruct <methods>;
    @javax.annotation.PreDestroy <methods>;
}

# Giữ lại các enum và giá trị của chúng
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# Giữ lại các tài nguyên cụ thể (ví dụ: tệp FXML)
-keepresourcefiles fxml/**.fxml
-keepresourcefiles application.properties
-keepresourcefiles application.yml

5. Đóng gói với jlink
jlink là một công cụ trong JDK cho phép bạn tạo một hình ảnh thời gian chạy tùy chỉnh chỉ chứa các mô-đun Java cần thiết cho ứng dụng của bạn. Điều này giúp giảm đáng kể kích thước ứng dụng và loại bỏ sự phụ thuộc vào JRE được cài đặt sẵn trên hệ thống của người dùng.

Các bước sử dụng jlink
Chuyển đổi dự án sang mô-đun (nếu chưa): Đối với các ứng dụng JavaFX + Spring Boot, bạn cần đảm bảo rằng dự án của mình được mô-đun hóa (sử dụng module-info.java) hoặc sử dụng các plugin xây dựng hỗ trợ mô-đun hóa tự động.

module-info.java ví dụ:

module com.yourcompany.yourapp {
    requires javafx.controls;
    requires javafx.fxml;
    requires spring.boot.autoconfigure;
    requires spring.context;
    requires spring.boot;
    requires spring.beans;
    requires spring.core;

    opens com.yourcompany.yourapp to javafx.fxml, spring.core, spring.beans, spring.context;
    opens com.yourcompany.yourapp.controller to javafx.fxml, spring.core, spring.beans, spring.context; // Mở các gói controller
    exports com.yourcompany.yourapp;
}

Tạo JAR có thể chạy được (hoặc JAR mô-đun): Đảm bảo bạn có một JAR chứa mã ứng dụng của mình và các phụ thuộc. Đối với Spring Boot, đây thường là một JAR có thể thực thi được.

Chạy lệnh jlink:

jlink --module-path <đường_dẫn_đến_các_mô_đun_javafx>:<đường_dẫn_đến_jar_ứng_dụng> --add-modules <mô_đun_ứng_dụng>,javafx.controls,javafx.fxml,spring.boot,spring.context --output <thư_mục_đầu_ra_hình_ảnh_runtime> --launcher <tên_launcher>=<mô_đun_ứng_dụng>/<lớp_chính>

<đường_dẫn_đến_các_mô_đun_javafx>: Đường dẫn đến các mô-đun JavaFX SDK (ví dụ: path/to/javafx-sdk-17/lib).

<đường_dẫn_đến_jar_ứng_dụng>: Đường dẫn đến JAR của ứng dụng của bạn (có thể là JAR đã được làm rối mã).

<mô_đun_ứng_dụng>: Tên mô-đun của ứng dụng của bạn (được định nghĩa trong module-info.java).

<tên_launcher>: Tên bạn muốn cho tệp thực thi.

<lớp_chính>: Lớp chính của ứng dụng JavaFX của bạn (ví dụ: com.yourcompany.yourapp.YourJavaFxApplication).

<thư_mục_đầu_ra_hình_ảnh_runtime>: Thư mục nơi hình ảnh thời gian chạy tùy chỉnh sẽ được tạo.

Lưu ý: Đối với ứng dụng Spring Boot, bạn có thể cần phải đóng gói nó như một JAR "truyền thống" (không phải JAR có thể thực thi được của Spring Boot) trước khi sử dụng jlink, hoặc sử dụng các plugin xây dựng tích hợp jlink để xử lý các phụ thuộc của Spring Boot một cách chính xác.

6. Đóng gói với jpackage
jpackage là một công cụ trong JDK để đóng gói các ứng dụng Java thành các gói dành riêng cho nền tảng (ví dụ: .exe cho Windows, .dmg cho macOS, .deb/.rpm cho Linux). Nó có thể sử dụng hình ảnh thời gian chạy được tạo bởi jlink.

Các bước sử dụng jpackage
Chuẩn bị tài nguyên: Chuẩn bị các tệp cần thiết như biểu tượng ứng dụng (.ico cho Windows, .icns cho macOS), tệp giấy phép, v.v.

Chạy lệnh jpackage:

jpackage --input <thư_mục_chứa_jar_ứng_dụng> --name "Tên ứng dụng của bạn" --main-jar <tên_jar_chính> --main-class <lớp_chính> --type <loại_gói> --dest <thư_mục_đầu_ra> --runtime-image <thư_mục_hình_ảnh_runtime_jlink> --icon <đường_dẫn_đến_biểu_tượng> --vendor "Tên công ty của bạn" --app-version "1.0.0" --copyright "Bản quyền (C) 2023 Tên công ty của bạn"

<thư_mục_chứa_jar_ứng_dụng>: Thư mục chứa JAR chính của ứng dụng của bạn.

<tên_jar_chính>: Tên của tệp JAR chính của ứng dụng.

<lớp_chính>: Lớp chính của ứng dụng JavaFX của bạn.

<loại_gói>: Loại gói (ví dụ: exe, dmg, deb, rpm).

<thư_mục_đầu_ra>: Thư mục nơi trình cài đặt sẽ được tạo.

<thư_mục_hình_ảnh_runtime_jlink>: Đường dẫn đến hình ảnh thời gian chạy được tạo bởi jlink.

<đường_dẫn_đến_biểu_tượng>: Đường dẫn đến tệp biểu tượng ứng dụng.

Ví dụ cho Windows (.exe):

jpackage --input target --name "MyJavaFXSpringBootApp" --main-jar my-app.jar --main-class com.yourcompany.yourapp.YourJavaFxApplication --type exe --dest dist --runtime-image my-runtime --icon src/main/resources/icon.ico --vendor "Your Company" --app-version "1.0.0" --copyright "Copyright (C) 2023 Your Company"

Lưu ý: jpackage yêu cầu các công cụ xây dựng gói gốc được cài đặt trên hệ thống của bạn (ví dụ: WiX Toolset cho Windows, Xcode Command Line Tools cho macOS).

7. Tích hợp với Maven/Gradle
Để tự động hóa toàn bộ quy trình, bạn nên tích hợp ProGuard, jlink và jpackage vào tệp pom.xml (Maven) hoặc build.gradle (Gradle) của mình.

Ví dụ Maven (pom.xml)
Việc tích hợp này khá phức tạp và thường yêu cầu nhiều plugin khác nhau (ví dụ: maven-shade-plugin để tạo JAR lớn, javafx-maven-plugin để hỗ trợ JavaFX và jlink/jpackage).

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version> <relativePath/> </parent>

    <groupId>com.yourcompany</groupId>
    <artifactId>your-javafx-spring-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>Your JavaFX Spring Boot App</name>

    <properties>
        <java.version>17</java.version>
        <javafx.version>21</javafx.version> <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <main.class>com.yourcompany.yourapp.YourJavaFxApplication</main.class>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency> -->

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

        <!-- <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-graphics</artifactId>
            <version>${javafx.version}</version>
            <classifier>win</classifier>
        </dependency>
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-graphics</artifactId>
            <version>${javafx.version}</version>
            <classifier>linux</classifier>
        </dependency>
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-graphics</artifactId>
            <version>${javafx.version}</version>
            <classifier>mac</classifier>
        </dependency> -->

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>${main.class}</mainClass>
                    <skip>true</skip>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                        <configuration>
                            <classifier>exec</classifier> <mainClass>${main.class}</mainClass>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>${main.class}</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>

            <plugin>
                <groupId>com.github.wvengen</groupId>
                <artifactId>proguard-maven-plugin</artifactId>
                <version>2.6.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>proguard</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <proguardInclude>${project.basedir}/proguard.pro</proguardInclude>
                    <obfuscate>true</object>
                    <injar>${project.build.finalName}.jar</injar>
                    <outjar>${project.build.finalName}-obfuscated.jar</outjar>
                    <libs>
                        <lib>${java.home}/jmods</lib>
                        <lib>${java.home}/lib/rt.jar</lib> <lib>${java.home}/lib/ext</lib>
                        <lib>${java.home}/lib</lib>
                        <lib>${project.basedir}/path/to/javafx-sdk-${javafx.version}/lib/javafx.base.jar</lib>
                        <lib>${project.basedir}/path/to/javafx-sdk-${javafx.version}/lib/javafx.controls.jar</lib>
                        <lib>${project.basedir}/path/to/javafx-sdk-${javafx.version}/lib/javafx.fxml.jar</lib>
                        <lib>${project.basedir}/path/to/javafx-sdk-${javafx.version}/lib/javafx.graphics.jar</lib>
                        <lib>${project.basedir}/path/to/javafx-sdk-${javafx.version}/lib/javafx.media.jar</lib>
                        <lib>${project.basedir}/path/to/javafx-sdk-${javafx.version}/lib/javafx.swing.jar</lib>
                        <lib>${project.basedir}/path/to/javafx-sdk-${javafx.version}/lib/javafx.web.jar</lib>
                    </libs>
                    <options>
                        -dontoptimize
                        -dontshrink
                        -verbose
                        -printmapping target/proguard-mapping.txt
                        -printseeds target/proguard-seeds.txt
                        -printusage target/proguard-usage.txt
                    </options>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>com.guardsquare</groupId>
                        <artifactId>proguard-base</artifactId>
                        <version>7.4.0</version> <scope>runtime</scope>
                    </dependency>
                </dependencies>
            </plugin>

            <plugin>
                <groupId>org.openjfx</groupId>
                <artifactId>javafx-maven-plugin</artifactId>
                <version>0.0.8</version> <configuration>
                    <mainClass>${main.class}</mainClass>
                    <jlinkImageName>my-app-runtime</jlinkImageName>
                    <jlinkZipName>my-app-runtime</jlinkZipName>
                    <noManPages>true</noManPages>
                    <noHeaderFiles>true</noHeaderFiles>
                    <stripDebug>true</stripDebug>
                    <launcher>MyJavaFXApp</launcher>
                    <appName>MyJavaFXSpringBootApp</appName>
                    <vendor>Your Company</vendor>
                    <appVersion>${project.version}</appVersion>
                    <copyright>Copyright (C) ${maven.build.timestamp} Your Company</copyright>
                    <icon>src/main/resources/icon.ico</icon> <runtimeImage>
                        <directory>target/jlink-runtime</directory>
                    </runtimeImage>
                    <jpackage>
                        <output>target/jpackage-output</output>
                        <type>exe</type> </jpackage>
                </configuration>
                <executions>
                    <execution>
                        <id>jlink</id>
                        <goals>
                            <goal>jlink</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>jpackage</id>
                        <goals>
                            <goal>jpackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

Lưu ý quan trọng cho Maven:

Bạn cần điều chỉnh các đường dẫn đến JavaFX SDK trong cấu hình ProGuard nếu bạn không sử dụng các phụ thuộc theo mô-đun hoặc nếu bạn đang chạy ProGuard độc lập.

Việc tích hợp Spring Boot với jlink và jpackage có thể phức tạp do cách Spring Boot đóng gói các phụ thuộc. javafx-maven-plugin có thể giúp đơn giản hóa điều này, nhưng bạn có thể cần thử nghiệm với các cấu hình khác nhau (ví dụ: tạo một JAR "thông thường" thay vì JAR có thể chạy được của Spring Boot, sau đó đưa nó vào jlink).

Đảm bảo rằng main.class của bạn trỏ đến lớp JavaFX Application của bạn.

Bạn cần cài đặt các công cụ xây dựng gói native (ví dụ: WiX Toolset cho Windows, Xcode Command Line Tools cho macOS) để jpackage hoạt động.

8. Kết luận
Bằng cách làm theo các bước này, bạn có thể tạo một ứng dụng JavaFX + Spring Boot được làm rối mã, được đóng gói trong một hình ảnh thời gian chạy tùy chỉnh và được phân phối dưới dạng trình cài đặt dành riêng cho nền tảng. Điều này giúp bảo vệ mã của bạn, giảm kích thước phân phối và cung cấp trải nghiệm cài đặt chuyên nghiệp cho người dùng cuối.

Hãy nhớ rằng việc gỡ lỗi cấu hình ProGuard có thể tốn thời gian. Luôn bắt đầu với các quy tắc keep tối thiểu và thêm dần khi bạn gặp lỗi ClassNotFoundException hoặc NoSuchMethodException trong quá trình chạy ứng dụng đã được làm rối mã.
