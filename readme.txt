<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Your project details -->
    <groupId>com.example</groupId>
    <artifactId>your-project</artifactId>
    <version>1.0.0</version>

    <properties>
        <spring.boot.version>2.5.3</spring.boot.version>
        <swagger.codegen.version>3.0.0</swagger.codegen.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Starter Web - Add other Spring Boot starters as per your project requirements -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>

        <!-- Swagger Codegen Maven Plugin -->
        <dependency>
            <groupId>io.swagger.codegen.v3</groupId>
            <artifactId>swagger-codegen-cli</artifactId>
            <version>${swagger.codegen.version}</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Maven Compiler Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <!-- Swagger Codegen Plugin -->
            <plugin>
                <groupId>io.swagger.codegen.v3</groupId>
                <artifactId>swagger-codegen-maven-plugin</artifactId>
                <version>${swagger.codegen.version}</version>
                <executions>
                    <execution>
                        <id>generate-spring</id>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                        <configuration>
                            <!-- Specify the location of your OpenAPI specification file -->
                            <inputSpec>src/main/resources/openapi.yaml</inputSpec>
                            <!-- Specify the output directory for the generated code -->
                            <output>${project.build.directory}/generated-sources/swagger</output>
                            <!-- Set the code generator to use Spring options -->
                            <generatorName>spring</generatorName>
                            <!-- Customize the generated API package and model package names -->
                            <apiPackage>com.example.api</apiPackage>
                            <modelPackage>com.example.model</modelPackage>
                            <!-- You can add more configuration options based on your requirements -->
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
