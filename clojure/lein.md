

使用atom，需要在它变化时，变化 需要在外层加(fn []) 

``` clojure
(defn yyy []
    (fn []
        (let [abc (:handler @!location)]
            [:div [abc]])))
```

不能是

``` clojure
(defn yyy []
    (let [abc (:handler @!location)]
        (fn []
            [:div [abc]])))
```

``` clojure
 (:import [goog.net XhrIo EventType WebSocket]
           [goog.net.xpc CfgFields CrossPageChannel]
           [goog Uri])
(.getParameterValue
                        (Uri. (.-href (.-location js/window)))
                        "xpc")
```

# maven使用lein

```xml
<build>
        <sourceDirectory>src</sourceDirectory>
        <resources>
            <resource>
                <directory>resources</directory>
            </resource>
        </resources>
        <plugins>
            <!-- 这个插件只有run一个 -->
            <plugin>
                <groupId>ingesolvoll</groupId>
                <artifactId>lein-maven-plugin</artifactId>
                <version>1.0</version>
                <executions>
                    <execution>
                        <id>compile-clojure</id>
                        <goals>
                            <goal>run</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!-- 这个插件有多种命令 -->
            <plugin>
                <groupId>com.theoryinpractise</groupId>
                <artifactId>clojure-maven-plugin</artifactId>
                <version>1.8.3</version>
                <executions>
                    <execution>
                        <id>compile</id>
                        <!--<phase>compile</phase>-->
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <sourceDirectories>
                        <sourceDirectory>src</sourceDirectory>
                    </sourceDirectories>
                </configuration>
            </plugin>
        </plugins>
    </build>
```



