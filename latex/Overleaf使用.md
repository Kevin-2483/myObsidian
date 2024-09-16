### 增强 Overleaf 实例编译环境

如果这时候你把很多期刊和会议的 Latex 模板扔到这个自建的 Overleaf 实例中编译，你会发现大部分情况下都是无法编译成功的。上面说到，overleaf.com 是一个开箱即用的集成化环境，而这个实例中仅包含编辑器部分和基本的 Latex 编译器。而两者之间的差别主要在于以下几个部分：

- Latex 宏包
- 系统字体
- TikZ & Syntax highlighter 的依赖
- TexLive 版本

手动把这几个部分的差别补足就是本文的重点。

**Latex 宏包。**Latex 对于学术排版比较好用的地方就是它对于学术内容的元素有着大量的宏包支持，比如伪代码等。宏包可以理解为 HTML/CSS 代码里的第三方库，只要调用相应的模板片段（接口）并填入自己的内容（参数），就能生成宏包原本定义好的漂亮的格式。

安装宏包并不困难，我们需要进入到 Overleaf 容器的内部，通过 TexLive 自带的包管理器 tlmgr 就能安装指定的宏包。但这里并不推荐单独安装缺少了宏包，因为 Latex 背后的宏包依赖层层嵌套且依赖关系不明确，最好的办法是用存储空间换省心，一次性把所有的宏包都安装了。

```shell
cd ./overleaf-toolkit
./bin/shell
tlmgr install scheme-full
```

Scheme-full 指代了 [CTAN](https://sspai.com/link?target=https%3A%2F%2Fctan.org%2Fpkg%2Flatex) 中所有的 Latex 宏包，大约需要至少 2G 的存储空间，整个下载过程非常的漫长，建议耐心等待。

**系统字体。**overleaf.com 中包含了[数百种字体](https://sspai.com/link?target=https%3A%2F%2Fwww.overleaf.com%2Flearn%2Flatex%2FQuestions%2FWhich_OTF_or_TTF_fonts_are_supported_via_fontspec%253F) 以供不同的模板无痛编译。但由于字体文件比较大且容易产生版权问题，因此需要我们自行安装。Overleaf 容器是以 Ubuntu 镜像为底层容器，因此字体的安装方法和 Ubuntu 一样，可以通过 Google 搜索很容易得到答案。

比如，我需要安装 Times New Roman 这个毕业论文格式中指定的字体，在 Windows 中自带但 Ubuntu 系统中没有。大部分常见的字体可以通过 Ubuntu 自带的 apt 包管理器安装，包的名称可以通过 Google 搜索获得。

```shell
cd ./overleaf-toolkit
./bin/shell
apt install ttf-mscorefonts-installer
```

对于少部分比较小众的字体，需要手动复制字体到 Overleaf 容器中，并刷新字体索引。

```shell
docker cp /path/to/your/font sharelatex:/usr/local/share/fonts/
cd ./overleaf-toolkit
./bin/shell
fc-cache -f -v
```

**TikZ & Syntax highlighter 的依赖。**TikZ 是 Latex 中画图的工具包，也是 Latex 强大的代表功能。TikZ 实际使用中会因为缺少 SVG 支持遇到一些错误，需要 inkscape 系统依赖。而 Syntax highlighter 是代码片段中高亮功能所需要的依赖，需要安装 python3-pygments 包。

```shell
cd ./overleaf-toolkit
./bin/shell
apt install inkscape python3-pygments
```

**(Optional) TexLive 版本。**还有一些编译错误是由于 TexLive 版本过低导致的，toolkit 一般会在 TexLive 新版本释放后更新容器的最新版本。toolkit 中自带了升级脚本，可以一键命令升级。

```shell
cd ./overleaf-toolkit
./bin/upgrade
```
