# 普通模式

配置里加入 :deploy-repositories（不配置这个，上传时会使用:repositories的配置）

``` clojure
;; ~/.lein/profiles.clj
{
  :user {
    :repositories [["me" {:url "https://dc.wisdragon.com/repository/maven-public/"
                          :snapshots {:update :always}}]]
    :plugin-repositories [["me" {:url "https://dc.wisdragon.com/repository/maven-public/"
                                 :snapshots {:update :always}}]]
    :deploy-repositories  [["snapshots" "https://dc.wisdragon.com/repository/maven-snapshots/"]
                           ["releases" "https://dc.wisdragon.com/repository/maven-releases/"]]
    }
}
```

> 使用提交命令`lein deploy` 后显示输入用户名，再输入密码，就可以提交到上面的仓库了。
>
> 使用 `lein deploy clojars` 提交到clojars 仓库，这个在lein2.0后就不需要配置了。

# 配置账户密码模式

1. 直接在每个:deploy-repositories单独配置用户密码，不太方便给clojars中心配置的用户密码。

```clojure
;; ~/.lein/profiles.clj
{
  :user {
    :repositories [["me" {:url "https://dc.wisdragon.com/repository/maven-public/"
                          :snapshots {:update :always}}]]
    :plugin-repositories [["me" {:url "https://dc.wisdragon.com/repository/maven-public/"
                                 :snapshots {:update :always}}]]
    :deploy-repositories  [["snapshots" {:url "https://dc.wisdragon.com/repository/maven-snapshots/"
                                         :username "username" :password "password"}]
                           ["releases" {:url "https://dc.wisdragon.com/repository/maven-releases/"
                                        :username "username" :password "password"
                                        :sign-releases false}]]
    }
}
```

> 配置`:sign-releases false`，上传releases版本时不需要给每个文件生成一份签名（sign  files with GPG，可以理解为水印，可以使用你的公钥验证文件，表明是我发出的文件）；如果你安装了gpg，生成了密钥可以不配置这个，否则需要加上这个配置参数。
>
> snapshots不需要配置:sign-releases false，snapshots版本上传时不签名。

2. 配置文件加入:auth 信息，同样使用命令上传则不在需要输入用户密码，此模式不安全，用户密码都明文记载。

```clojure
;; ~/.lein/profiles.clj
{
  :user {
    :repositories [["me" {:url "https://dc.wisdragon.com/repository/maven-public/"
                          :snapshots {:update :always}}]]
    :plugin-repositories [["me" {:url "https://dc.wisdragon.com/repository/maven-public/"
                                 :snapshots {:update :always}}]]
    :deploy-repositories  [["snapshots" "https://dc.wisdragon.com/repository/maven-snapshots/"]
                           ["releases" "https://dc.wisdragon.com/repository/maven-releases/"]]
    }
  :auth {
    :repository-auth {#"clojars" {:username "username"
                                  :password "password"
                                  :sign-releases false}
                      #"https://dc.wisdragon.com/.*" {:username "username"
                                                      :password "password"
                                                      :sign-releases false}}}
}
```

# 使用GPG模式

## gpg安装（mac不带gpg）

``` sh
# 安装gpg（先安装Homebrew）
brew install gnupg2
# 生成密钥-默认设置-有期限
gpg --generate-key
# 生成密钥-完整功能的密钥对生成，选择 1-2048-0-y-loong-582538746@qq.com- 。（推荐这个，可以设置无限期,创建时需要输入密码，可以不输入直接回车键，省去以后解密需要输入密码）
gpg --full-generate-key
# 查看公钥
gpg -k
# 查看私钥
gpg -K
# 删除公钥（先删除私钥）(ID可以是密钥ID，也可以是用户ID，也可以直接用用户名loong)
gpg --delete-key [ID]
# 删除私钥
gpg --delete-secret-keys [ID]
# 导出公钥（可使用-a参数为输出为ASCII字符格式，不带-a为二进制格式）
gpg -a -o public.key.asc --export [ID]
# 导出私钥（私钥请妥善保管）
gpg -a -o private.key.asc --export-secret-keys [ID]
# 导入公钥
gpg --import public.key.asc
# 导入私钥（可以带上 --allow-secret-key-import 给予允许添加私钥的权限）
gpg --import private.key.asc
# 给lein密码文件加密
gpg --default-recipient-self -e \
    ~/.lein/credentials.clj > ~/.lein/credentials.clj.gpg
# 解密查看文件
gpg -d ~/.lein/credentials.clj.gpg
```

安装gpg后碰到的问题：

- 生成密钥时的弹窗以及其他弹窗都是乱码，安装gpgtools工具后显示正常。
- 使用idea工具上传时，遇到无法解密，因为密钥解密时需要输入密码，idea无法弹出输入密码框，安装gpgtools后可正常弹窗。（也可以通过在创建密钥时不使用密码来解决）

> 总结：使用完整密钥生成命令生成密钥，设置无限期和没有密码。再导出公钥和私钥保存，以后重装系统再重新导入就可以了，也不怎么需要安装gpgtools工具。
>
> 注意：==私钥妥善保管==，公钥可上传公钥服务器等，可给别人导入，别人用你的公钥加密文件传给你，你可以解密。同样你也可以导入别人的公钥加密文件，传给他后他可以解密。

## lein gpg配置

配置文件需要在每个仓库配置里加上`:creds :gpg`，同样使用上传的命令上传时自动解密不在需要输入用户密码。

```clojure
;; ~/.lein/profiles.clj
{
  :user {
    :repositories [["me" {:url "https://dc.wisdragon.com/repository/maven-public/"
                          :snapshots {:update :always}}]]
    :plugin-repositories [["me" {:url "https://dc.wisdragon.com/repository/maven-public/"
                                 :snapshots {:update :always}}]]
    :deploy-repositories  [["snapshots" {:url "https://dc.wisdragon.com/repository/maven-snapshots/" :creds :gpg}]
                           ["releases" {:url "https://dc.wisdragon.com/repository/maven-releases/"
                           :creds :gpg
                           ; :signing {:gpg-key "loong"}
                           }]]
    ; :signing {:gpg-key "EC6A18BDEB92C0BA"}
    :signing {:gpg-key "loong"}
    }
}
```

> :signing可以不配置，则使用gpg默认密钥解密，可以放到:user下，也可对每个仓库单独配置

创建credentials.clj文件保存账户密码，在使用上面gpg命令加密成~/.lein/credentials.clj.gpg，之后就可删除credentials.clj了。

```clojure
;; ~/.lein/credentials.clj
{
    #"clojars" {:username "username" :password "password"}
    #"https://dc.wiseloong.com/.*" {:username "username" :password "password"}
}
```

