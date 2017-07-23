---
title: Install Octave on Mac Sierra
date: 2017-07-23 18:36:42
tags:
- Machine Learning
- Homebrew
- Mac
- Octave
---

# Learning Machine Learning

I've joined ViSenze for almost half a year, and our CTO said we (me and Terry) should learn some machine learning knowledge, since there are a lot of Phd here in our company:). He recommended a course on Coursera - [Neural Networks for Machine Learning](https://www.coursera.org/learn/neural-networks).

I am facing the Quiz of Week 3, while it needs some coding works in Octave. 

# Problems

The Macbook, which lays in my home, hasn't been used for quite a time. When I use `brew` to install Octave, some weird issue happened:

``` bash
Error: uninitialized constant Formulary::FormulaNamespacefe4ce29a01455f41d6d0b08c39f76615::Octave::DevelopmentTools
Please report this bug:
    https://git.io/brew-troubleshooting
/usr/local/Library/Taps/homebrew/homebrew-science/octave.rb:31:in `<class:Octave>'
/usr/local/Library/Taps/homebrew/homebrew-science/octave.rb:1:in `load_formula'
/usr/local/Library/Homebrew/formulary.rb:21:in `module_eval'
/usr/local/Library/Homebrew/formulary.rb:21:in `load_formula'
/usr/local/Library/Homebrew/formulary.rb:38:in `load_formula_from_path'
/usr/local/Library/Homebrew/formulary.rb:87:in `load_file'
/usr/local/Library/Homebrew/formulary.rb:78:in `klass'
/usr/local/Library/Homebrew/formulary.rb:74:in `get_formula'
/usr/local/Library/Homebrew/formulary.rb:171:in `get_formula'
/usr/local/Library/Homebrew/formulary.rb:211:in `factory'
/usr/local/Library/Homebrew/extend/ARGV.rb:18:in `block in formulae'
/usr/local/Library/Homebrew/extend/ARGV.rb:16:in `map'
/usr/local/Library/Homebrew/extend/ARGV.rb:16:in `formulae'
/usr/local/Library/Homebrew/cmd/install.rb:95:in `install'
/usr/local/Library/brew.rb:87:in `<main>'
```

# Solution

I found there is guide on Octave [Wiki](http://wiki.octave.org/Octave_for_MacOS_X#Simple_Installation_Instructions_2). Let me summarize it:

``` bash
sudo chown -R $(whoami) /usr/local
brew update && brew upgrade # it will take some time
brew cask install xquartz
brew install octave # it will take a lot of time...
```
# After brew install octave

When I typed 'octave' in my terminal, there is an issue like:

``` bash
$octave
dyld: Library not loaded: /usr/local/opt/suite-sparse/lib/libsuitesparseconfig.4.5.4.dylib
  Referenced from: /usr/local/bin/octave
  Reason: image not found
Abort trap: 6
```

And actually I've already installed `suite-sparse`, version 4.5.5. You need to create a soft link:

``` bash
cd /usr/local/opt/suite-sparse/lib/
ln -s libsuitesparseconfig.4.5.5.dylib libsuitesparseconfig.4.5.4.dylib
```

# Create your application

It is easier if we can launch octave in our Application, not in terminal. We can use `Script Editor` in macOS:

``` applescript
tell application "Terminal"	do script "`which octave`; exit"end tell
```

# Screenshot

![Alt text](/images/octave.png)