# CLI Overview

```powershell
###########################################################
# Windows specific command prompt instructions shown here #
###########################################################
```

```shell
#########################################################
# macOS specific command prompt instructions shown here #
#########################################################
```

The Bonsai Command Line Interface (CLI) is a text-based tool that enables you to configure and control the Bonsai Artificial Intelligence Engine. The CLI is especially useful for automation and connection to other tools. Currently, there are some actions that can only be performed using the CLI, such as editing a BRAIN's project file and getting predictions from a BRAIN.

This guide will walk you through any prerequisites you may need, and how to install the Bonsai CLI. The reference document for all of the commands you will see in this guide can be found in the [CLI Reference][1].



# Install Prerequisites

Before you begin, you will need to have access to the Bonsai Platform preview. If you don't have access yet, request access at [bons.ai][2].

The next section walks you through how to install Git, Python, and pip, but if you're already ahead of the curve and **have them all installed**, skip down to [Install the Bonsai CLI][3]. 

<aside class="notice">
We highly recommend macOS users download Python directly in the Manual Install instructions or use <a href="http://brew.sh/">Homebrew</a> so you can perform the following steps on user-land Python rather than <a href="https://github.com/MacPython/wiki/wiki/Which-Python">system Python</a>.
</aside>

**There are three sets of install instructions, please pick the one that suits your situation:**

1. If you have **Anaconda** installed on your system, skip down to [Install with Anaconda][9].
2. If you have **Chocolatey (Windows)** or **Homebrew (macOS)** installed on your system, skip down to [Install with Chocolatey or Homebrew][7].
3. If you *do not* have Chocolatey, Homebrew, or Anaconda installed on your system continue to follow our **Manual Install** intructions below.

<aside class="notice">
The two tabs to the right allow you to follow this guide as a macOS or Windows user. Instructions also vary slightly between Python 2 and Python 3 so please be sure to follow your version's instructions.
</aside>

## Manual Install

### Windows

* **Git**: Download and install Git from [Git for Windows][11]. 
* **Python**: Download and install Python (3.6 or 2.7) from [python.org][4].

<aside class="warning">
For users installing <b>Python 2.7</b>: Make sure to include adding Python to your PATH when customizing your install on the third screen. <br>
For users installing <b>Python 3.6</b>: Make sure to check the box on the first screen of the install to add Python to your PATH.
</aside>

* **pip**: Python version 2.7.9 and greater come with pip, but if for some reason you need to use a different version of Python please follow [these instructions][12].

<aside class="notice">
Please remember to reboot your computer after installing Python and Git to make sure all install settings take effect.
</aside>

**Great!** Now that you have Git, Python, and pip installed, you can skip down to [Install the Bonsai CLI][3]. 

### macOS

* **Git**: Git is more than likely already installed on your computer if you have Xcode, but if not, run `git` from your command prompt and the system will prompt you to install.
* **Python**: Download and install Python (3.6 or 2.7) from [python.org][4].

<aside class="notice">
We highly recommend macOS users download Python directly so you can follow this guide on user-land Python rather than <a href="https://github.com/MacPython/wiki/wiki/Which-Python">system Python</a>.
</aside>

* **pip**: Python version 2.7.9 and greater come with pip (or pip3 for version 3+), but if for some reason you need to use a different version of Python please follow [these instructions][12].

**Great!** Now that you have Git, Python, and pip installed, you can skip down to [Install the Bonsai CLI][3]. 

## Install with Chocolatey or Homebrew

If you *do not* have Chocolatey or Homebrew installed on your system, please go to the [Manual Install][15] or [Install with Anaconda][9] instructions.

> Python 2

```powershell
choco install git
choco install python2
```
```shell
brew install python
```

> Python 3

```powershell
choco install git
choco install python
```
```shell
brew install python3
```

### Windows

**If you're already familiar with [Chocolatey][14], follow these instructions.**

<aside class="notice">
Please remember to run choco commands in administrator powershell or command prompt.
</aside>

### macOS

**If you're already familiar with [Homebrew][6], follow these instructions.**

* **Git**: Git is more than likely already installed on your computer if you have Xcode, but if not, run `git` from your command prompt and the system will prompt you to install.
* **Python**: Run the install command on the tab to the right to `brew install` the Python version of your choice.

## Install with Anaconda

```
conda install git
```

If you *do not* have Anaconda installed on your system, please go to the [Manual Install][15] or [Install with Chocolatey or Homebrew][7] instructions.

`conda` is a tool for installing and managing Python and R dependencies. Anaconda includes Python and pip, so all you need to do is install Git if you haven't already. For more information, refer to the [Anaconda website][10].



# Install the Bonsai CLI

> Python 2 and Anaconda

```
pip install bonsai-cli
bonsai configure # for authentication
```

> Python 3

```powershell
# same as above
```
```shell
pip3 install bonsai-cli
bonsai configure # for authentication
```

From your command prompt, install the Bonsai Command Line Interface tool and configure (authenticate) it. [`bonsai configure`][17] will ask for your Access Key. You can find your Access Key in your [Bonsai Account Settings][8].



# Next Steps

Now that you've installed the Bonsai CLI you can [Use the CLI for Training][5] tutorial to get started with the Bonsai Platform.

If you have already trained a BRAIN on the web and are looking to use the Bonsai CLI to get predictions from your BRAIN you can either follow the instructions on the web to connect your local environment to your BRAIN or see [Predict with Your BRAIN][16] for an example.

[1]: ../references/cli-reference.html
[2]: https://bons.ai/get-started
[3]: #install-the-bonsai-cli
[4]: https://www.python.org
[5]: local-dev-guide.html
[6]: http://brew.sh/
[7]: #install-with-chocolatey-or-homebrew
[8]: https://beta.bons.ai/accounts/key
[9]: #install-with-anaconda
[10]: https://www.anaconda.com/what-is-anaconda/
[11]: https://git-for-windows.github.io/
[12]: https://pip.pypa.io/en/stable/installing/
[13]: https://github.com/MacPython/wiki/wiki/Which-Python
[14]: https://chocolatey.org/
[15]: #manual-install
[16]: local-dev-guide.html#predict-with-your-brain
[17]: ../references/cli-reference.html#bonsai-configure