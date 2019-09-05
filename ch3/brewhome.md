# BrewHome-Mac命令行包管理工具

- ## 安装 Homebrew

  

  ```
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```

  将以上命令粘贴至终端。

  脚本会在执行前暂停，并说明它将做什么。高级安装选项在 [这里](https://docs.brew.sh/Installation) (required for Linux and Windows Subsystem for Linux)。

- ## Homebrew 能干什么?

  

  使用 Homebrew 安装 Apple（或您的 Linux 系统）没有预装但 [你需要的东西](https://formulae.brew.sh/formula/)。

  ```
  $ brew install wget
  ```

- Homebrew 会将软件包安装到独立目录，并将其文件软链接至 `/usr/local` 。

  ```
  $ cd /usr/local
  $ find Cellar
  Cellar/wget/1.16.1
  Cellar/wget/1.16.1/bin/wget
  Cellar/wget/1.16.1/share/man/man1/wget.1
  
  $ ls -l bin
  bin/wget -> ../Cellar/wget/1.16.1/bin/wget
  ```

- Homebrew 不会将文件安装到它本身目录之外，所以您可将 Homebrew 安装到任意位置。

- 轻松创建你自己的 Homebrew 包。

  ```
  $ brew create https://foo.com/bar-1.0.tgz
  Created /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/bar.rb
  ```

- 完全基于 Git 和 Ruby，所以自由修改的同时你仍可以轻松撤销你的变更或与上游更新合并。

  ```
  $ brew edit wget # 使用 $EDITOR 编辑!
  ```

- Homebrew 的配方都是简单的 Ruby 脚本：

  ```
  class Wget < Formula
    homepage "https://www.gnu.org/software/wget/"
    url "https://ftp.gnu.org/gnu/wget/wget-1.15.tar.gz"
    sha256 "52126be8cf1bddd7536886e74c053ad7d0ed2aa89b4b630f76785bac21695fcd"
  
    def install
      system "./configure", "--prefix=#{prefix}"
      system "make", "install"
    end
  end
  ```

- Homebrew 使 macOS（或您的 Linux 系统）更完整。使用 `gem` 来安装 RubyGems、用 `brew` 来安装那些依赖包。

- “要安装，请拖动此图标......”不会再出现了。使用 `brew cask` 安装 macOS 应用程序、字体和插件以及其他非开源软件。

  ```
  $ brew cask install firefox
  ```

- 制作一个 cask 就像创建一个配方一样简单。

  ```
  $ brew cask create foo
  Editing /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask/Casks
  ```



https://brew.sh/index_zh-cn