---
layout: post
title: "Conda"
category: "其他"
order: "1"
---

大部分人对 conda 的第一印象是：它是一个管理 Python 包的包管理工具。那么， conda 和 Python 的官方包管理器 pip 有什么区别？Anaconda, conda-forge, Miniconda, Miniforge 又都是什么？本文将解释这些东西的来龙去脉。

为了管理包和项目环境，Python 的官方做法是 pip + venv。这一方案的缺点是，pip 对于跨语言依赖的包支持地很差，而跨语言依赖的包对于 Python 越来越重要了。比如我们熟悉的 Numpy, Scipy, pandas 等库，就是使用 C 等语言编写，然后加一层封装让 Python 代码可以直接调用这些程序。这样一来，使用者就既能享受 Python 的简单语法，又能解决 Python 解释器性能低下的问题。所以，这些包吸引了大量使用者用来做科学计算，但是用当时的 pip 还很难直接一键 pip install 安装，这就成了问题。

当时的 Continuum Analytics 公司为了解决这个问题，将 Python 和常用的数学库打包成了一个超大的安装包，称为 Anaconda Distribution。最初的 Anaconda Distribution 还将科学计算中常用的 R 语言也包含了进去。而当下的 Anaconda 已经支持绝大多数主流编程语言。

这样，用户就纷纷转向了 Anaconda。尽管当前官方的 pip 也能很好地解决这些包的安装了，但是路径依赖使得 conda 还是拥有大部分的用户。现在的 AI 项目几乎全部是用 conda 来管理 Python 依赖库的。而且，由于 Anaconda 过于成功，这个公司已经改名为 Anaconda 公司。

最初的 Anaconda 只是一个语言和依赖库的整合包。它的体积相当大，而且用户难以定制一个自己的整合包。也就是，用户无法向其中增加和删除包。更严重的是，科学计算库常常会更新，而 Anaconda 没有为用户提供自动更新服务。因此，Anaconda 中就加入了现在我们熟悉的 conda 命令。conda install, conda uninstall 就可以方便地下载和删除包。

Anaconda 做的更好的是，它将这些包维护成了一个软件仓库。对于那些底层使用 C 等语言的包，Anaconda 为不同的操作系统提供不同的版本，并精心处理了需要的依赖项，放在这个仓库中。这样一来，conda install 往往可以做到开箱即用。

为了解决项目环境隔离的问题，Anaconda 也使用虚拟环境机制。使用 conda create -n \<name> 就可以创建一个虚拟环境，使用 conda activate \<name> 激活这个环境，就可以安装和使用虚拟环境以及其中的包了。如果没有激活环境，你就会处于 conda 的默认环境 base 里。但是最好不要随意将东西安装到 base 环境里，因为这是 conda 本身运行的环境，如果发生故障会影响到整个 conda。

为了将更多的包加入到 Anaconda 维护的软件仓库中，Anaconda 公司希望允许开发者直接上传他们的包，并由开发者自行维护。这个平台是 Anaconda.org，用户可以在上面创建自己的软件仓库，维护包并处理兼容性问题。这样的一个仓库就是一个 channel。Anaconda 公司自身维护的软件仓库是 defaults channel。准确来说，这其实是三个 channel：main(大多数软件包), R(R相关包), msys2(兼容Windows的Linux工具)。当我们执行 conda install 的时候，默认的下载源就是 defaults。

有了 Anaconda.org 之后，实际上用户就不需要下载安装整个 Anaconda 了，毕竟它的体积还是很大，而且其中的 90% 以上的内容都不会使用到。用户只需要 conda 命令来自行安装需要的包就可以。这就是 Miniconda。Miniconda 也是由 Anaconda 公司提供的，只包括了 conda 命令工具和自身需要的基本依赖，因此体积小了很多。当然，现在的 Anaconda 还多了一些别的工具，例如图形化的包管理工具 Anaconda Navigator，Miniconda 就无法使用这个工具。

conda-forge，就是开源社区在 Anaconda.org 上维护的开源 channel。用户可以申请上传，审核通过之后就可以由所有人下载了。conda-forge 已经超过了 defaults，成为了最大的 channel。它的优点主要在于软件包数量多，更新速度快，不过它的兼容性可能略逊于 Anaconda 公司维护的仓库，不过也足以应对绝大多数场景。重要的是，2020 年之后，Anaconda 开始对商业用途的 defaults 下载进行收费，因此 conda-forge 越来越有取代 defaults 的趋势。在 conda 命令中添加一个 -c 参数，就可以指定下载包所用的 channel；或者可以修改配置文件，直接修改默认 channel，将它从 defaults 修改为你需要的。

虽然 Anaconda 公司对 defaults 的商业用途收费了，但 conda 工具本身是开源的。因此，开源社区自己打造了一个 Miniconda，也就是 Miniforge。它和 Miniconda 的区别就是默认 channel 不同，前者是 conda-forge 而后者是 defaults。

最后，为了提高 conda 这个工具本身的性能，开源社区还开发出了另一个工具 Mamba。它将 conda 中运行最慢的部分，例如依赖求解器用 C++ 重新实现了，并支持了多线程下载。Mamba 被默认包含在了 Miniforge 的安装包里，不需要额外安装。它的用法与 conda 基本相同，只需要将 conda 命令替换为 mamba 命令即可。 


内容来自：[15分钟彻底搞懂！Anaconda Miniconda conda-forge miniforge Mamba](https://www.bilibili.com/video/BV1Fm4ZzDEeY/)