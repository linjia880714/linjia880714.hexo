---
title: IntelliJ GUI Designer 打成可执行jar包
date: 2017-08-28 17:20:05
tags: IT工具
categories:
- IT
- java
- IDE
---

参考 [Problems with maven while using intelij idea gui designer](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206200489-Problems-with-maven-while-using-intelij-idea-gui-designer-?sort_by=votes)

报错：
```java
java.awt.IllegalComponentStateException: contentPane cannot be set to null.
        at javax.swing.JRootPane.setContentPane(JRootPane.java:621)
        at javax.swing.JFrame.setContentPane(JFrame.java:698)
        at com.jia.autocode.Panel.<init>(Panel.java:93)
        at com.jia.autocode.Main.main(Main.java:16)
```

IntelliJ GUI Designer的.form是使用自己的javac2 compiler，所以标准JDK是无法解析编译这个.form文件的。

pom.xml增加下面两个插件
1. maven-shade-plugin把项目打成一个可执行jar包
2. maven-antrun-plugin执行javac2 compiler

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.jia.autocode.Main</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>


<plugin>
    <artifactId>maven-antrun-plugin</artifactId>
    <version>1.7</version>
    <dependencies>
        <dependency>
            <groupId>com.sun</groupId>
            <artifactId>tools</artifactId>
            <version>1.8.0</version>
            <scope>system</scope>
            <systemPath>${java.home}/../lib/tools.jar</systemPath>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <phase>compile</phase>
            <configuration>
                <tasks>
                    <property name="IDEA.home" value="/Applications/IntelliJ IDEA.app/Contents"/> <!-- My IDEA home. -->
                    <path id="j2cp">
                        <pathelement location="${IDEA.home}/lib/asm-5.0.3.jar"/>
                        <pathelement location="${IDEA.home}/lib/asm-all.jar"/>
                        <pathelement location="${IDEA.home}/lib/javac2.jar"/>
                        <pathelement location="${IDEA.home}/lib/jdom.jar"/>
                        <pathelement location="${IDEA.home}/lib/jgoodies-forms.jar"/>
                    </path>
                    <path id="j2sp">
                        <pathelement location="${project.basedir}\src\main\java"/> 
                    </path>
                    <taskdef name="javac2" classpathref="j2cp" classname="com.intellij.ant.Javac2"/>
                    <javac2 destdir="${project.basedir}\target\classes"> 
                        <src refid="j2sp"/>
                    </javac2>
                </tasks>
            </configuration>
            <goals>
                <goal>run</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

请注意：
1. maven-shade-plugin的mainClass，请填写自己main方法所在的class
2. maven-antrun-plugin中的"IDEA.home"是IntelliJ所在的路径
3. ${IDEA.home}/lib/asm-5.0.3.jar，${IDEA.home}/lib/asm-all.jar等5个jar包，请到具体路径下看是否存在，名字有可能是不一样的