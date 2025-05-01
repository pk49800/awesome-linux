# Ubuntu 24.04 系统优化配置指南

## 一、切换软件源（提升下载速度）

```bash
sudo sed -i 's|http://.*archive.ubuntu.com|https://mirrors.aliyun.com|g' /etc/apt/sources.list
sudo sed -i 's|http://.*security.ubuntu.com|https://mirrors.aliyun.com|g' /etc/apt/sources.list
sudo apt update && sudo apt upgrade -y
```

**作用**：使用国内镜像源加速软件下载，示例为阿里云镜像源（可根据地理位置替换为清华、腾讯等源）

## 二、终端环境配置

### 1. 安装基础工具

```bash
sudo apt install -y git vim curl wget
```

### 2. 安装ZSH及其增强组件

```bash
# 安装ZSH
sudo apt install -y zsh

# 设为默认shell（需重新登录生效）
chsh -s $(which zsh)

# 安装Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 安装插件
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
sudo apt install -y autojump

# 编辑配置文件
vim ~/.zshrc
```

修改插件配置为：

```bash
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
  z
  autojump
)
```

**插件说明**：

- `zsh-autosuggestions`：输入命令时显示历史建议
- `zsh-syntax-highlighting`：命令语法高亮
- `z`：目录快速跳转
- `autojump`：学习用户习惯实现智能跳转

### 3. 终端美化配置

```bash
# 安装Powerline字体

# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts

# 刷新字体缓存
fc-cache -fv

# 修改主题
sed -i 's/ZSH_THEME=.*/ZSH_THEME="agnoster"/' ~/.zshrc

# 隐藏命令提示符前的用户名和主机名
echo "DEFAULT_USER=$USER" >> ~/.zshrc
```

**操作说明**：

1. 在终端首选项中将字体设置为 "Meslo LG S DZ for PowerLine"
2. 主题`agnoster`需要Powerline字体支持
3. `DEFAULT_USER`设置可隐藏冗余的用户名显示

## 三、系统字体配置

### 1. 安装Noto字体家族(默认已自带)

```bash
sudo apt install -y fonts-noto-cjk fonts-noto-color-emoji
```

### 2. 创建字体配置文件

```bash
mkdir -p ~/.config/fontconfig
vim ~/.config/fontconfig/font.conf
```

添加以下内容：

```xml
<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM 'urn:fontconfig:fonts.dtd'>
<fontconfig>
  <!--rendering options-->
  <match target="font">
    <edit name="autohint" mode="assign">
      <bool>false</bool>
    </edit>
    <edit name="hinting" mode="assign">
      <bool>true</bool>
    </edit>
    <edit name="hintstyle" mode="assign">
      <const>hintslight</const>
    </edit>
    <edit name="antialias" mode="assign">
      <bool>true</bool>
    </edit>
    <edit name="lcdfilter" mode="assign">
      <const>lcddefault</const>
    </edit>
    <edit name="rgba" mode="assign">
      <const>rgb</const>
    </edit>
  </match>

  <!-- no embeddedbitmap except for Twemoji -->
  <match target="font">
    <test name="family" compare="not_eq">
      <string>Twemoji</string>
    </test>
    <edit name="embeddedbitmap" mode="assign">
      <bool>false</bool>
    </edit>
  </match>
  <!-- no antialias for Twemoji
    or in italic in gvim it shows blank
    qual="all", or firefox uses web font non-aliased for https://status.python.org/
    specific to gvim, or Google Chrome uses web font non-aliased, e.g. home page, github.
  -->
  <match target="font">
    <test name="prgname" compare="eq">
      <string>gvim</string>
    </test>
    <test name="family" compare="eq">
      <string>Twemoji</string>
    </test>
    <edit name="antialias" mode="assign">
      <bool>false</bool>
    </edit>
  </match>

  <!-- includes -->
  <!-- include prefix="xdg">fontconfig/web-ui-fonts.conf</include -->

  <!-- Replace Source Han -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>Source Han Sans</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Noto Sans CJK SC</string>
    </edit>
  </match>
  <match target="pattern">
    <test qual="any" name="family">
      <string>Source Han Serif</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Noto Serif CJK SC</string>
    </edit>
  </match>

  <!-- Replace Noto Color Emoji -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>Noto Color Emoji</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Twemoji</string>
    </edit>
  </match>

  <!-- Default system-ui fonts -->
  <match target="pattern">
    <test name="family">
      <string>system-ui</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>sans-serif</string>
    </edit>
  </match>

  <!-- Default sans-serif fonts-->
  <match target="pattern">
    <test name="family">
      <string>sans-serif</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Noto Sans CJK SC</string>
      <string>Noto Sans</string>
      <string>Twemoji</string>
      <string>Symbols Nerd Font</string>
    </edit>
  </match>

  <!-- Default serif fonts-->
  <match target="pattern">
    <test name="family">
      <string>serif</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Noto Serif CJK SC</string>
      <string>Noto Serif</string>
      <string>Twemoji</string>
      <string>Symbols Nerd Font</string>
    </edit>
  </match>

  <!-- Default monospace fonts-->
  <match target="pattern">
    <test name="family">
      <string>monospace</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Noto Sans Mono CJK SC</string>
      <string>Symbols Nerd Font</string>
      <string>Twemoji</string>
      <string>Symbols Nerd Font</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="family" compare="contains">
      <string>Source Code</string>
    </test>
    <edit name="family" binding="strong">
      <string>Fira Code</string>
    </edit>
  </match>

  <!-- Replace english fonts-->
  <match target="pattern">
    <test name="prgname" compare="not_eq">
      <string>msedge</string>
    </test>
    <test name="family" compare="contains">
      <string>Noto Sans Mono CJK</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Fira Code</string>
    </edit>
  </match>

  <!-- Replace fonts for Chinese (Hong Kong) -->
  <match target="pattern">
    <test name="lang">
      <string>zh-HK</string>
    </test>
    <test name="family">
      <string>Noto Sans CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Sans CJK HK</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="lang">
      <string>zh-HK</string>
    </test>
    <test name="family">
      <string>Noto Serif CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <!-- not have HK -->
      <string>Noto Serif CJK TC</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="lang">
      <string>zh-HK</string>
    </test>
    <test name="family">
      <string>Noto Sans Mono CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Sans Mono CJK HK</string>
    </edit>
  </match>

  <!-- Replace fonts for Chinese (Taiwan) -->
  <match target="pattern">
    <test name="lang">
      <string>zh-TW</string>
    </test>
    <test name="family">
      <string>Noto Sans CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Sans CJK TC</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="lang">
      <string>zh-TW</string>
    </test>
    <test name="family">
      <string>Noto Serif CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Serif CJK TC</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="lang">
      <string>zh-TW</string>
    </test>
    <test name="family">
      <string>Noto Sans Mono CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Sans Mono CJK TC</string>
    </edit>
  </match>

  <!-- Replace fonts for Japanese -->
  <match target="pattern">
    <test name="lang">
      <string>ja</string>
    </test>
    <test name="family">
      <string>Noto Sans CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Sans CJK JP</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="lang">
      <string>ja</string>
    </test>
    <test name="family">
      <string>Noto Serif CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Serif CJK JP</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="lang">
      <string>ja</string>
    </test>
    <test name="family">
      <string>Noto Sans Mono CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Sans Mono CJK JP</string>
    </edit>
  </match>

  <!-- Replace fonts for Korean -->
  <match target="pattern">
    <test name="lang">
      <string>ko</string>
    </test>
    <test name="family">
      <string>Noto Sans CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Sans CJK KR</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="lang">
      <string>ko</string>
    </test>
    <test name="family">
      <string>Noto Serif CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Serif CJK KR</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="lang">
      <string>ko</string>
    </test>
    <test name="family">
      <string>Noto Sans Mono CJK SC</string>
    </test>
    <edit name="family" binding="strong">
      <string>Noto Sans Mono CJK KR</string>
    </edit>
  </match>

  <!-- 解决全角引号 -->
  <match target="pattern">
    <test name="lang" compare="contains">
      <string>en</string>
    </test>
    <test name="family" compare="contains">
      <string>Noto Sans CJK</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Noto Sans</string>
    </edit>
  </match>
  <match target="pattern">
    <test name="lang" compare="contains">
      <string>en</string>
    </test>
    <test name="family" compare="contains">
      <string>Noto Serif CJK</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Noto Serif</string>
    </edit>
  </match>

</fontconfig>

```

执行`fc-cache -fv`刷新字体缓存

## 四、GNOME桌面优化

### 1. 安装扩展

```bash
sudo apt install -y gnome-tweaks  chrome-gnome-shell
```

### 2. 通过Gnome Tweaks调整字体

- 界面文本：Noto Sans CJK SC Black
- 文档文本：Noto Sans CJK SC Bold
- 等宽文本：Noto Sans Mono CJK SC Bold

### 3. 安装推荐扩展

1. 从谷歌商店安装`gnome-shell-extensions` 插件管理器
2. 通过插件管理器搜索安装以下扩展：
- Customize IBus （iBUs输入法配置）
- Touchpad Gesture Customization （启用4指手势控制app行为，可代替Wayland不支持的双指横扫手势）
- Hide Top Bar （隐藏顶部状态栏）

## 四、快捷键修改和设置

1. 修改全局关闭窗口快捷键为Ctrl + Q
2. 修改音频切换上一/下一曲为Ctrl + Super + $\uparrow$/$\downarrow$

## 五、推荐补充配置

### 1. 系统性能优化

```bash
# 启用ZRAM
sudo apt install -y zram-config
```

### 2. 开发环境准备

```bash
# 安装基础编译环境
sudo apt install -y build-essential libssl-dev libffi-dev python3-dev

# 安装常用工具
sudo apt install -y neofetch htop tree ripgrep bat
```

### 3. 安全加固（可选）

```bash
# 配置防火墙
sudo ufw enable
sudo ufw default deny incoming
sudo ufw allow ssh
```

---

**注意事项**：

1. 所有.zshrc修改后需执行`source ~/.zshrc`生效
2. GNOME扩展安装可能需要先安装`gnome-shell-extension-manager`
3. 触摸板手势配置需要根据实际硬件调整灵敏度参数
4. 建议在配置前后创建系统快照以便回滚

以上配置可根据实际需求选择性实施，部分配置可能需要重启系统后才能完全生效。

## 六、安装常用软件

### 1. 配置flathub
```bash
# 安装flatpak
sudo apt install flatpak
# 添加flatpak的软件仓库flathub
flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo
#更换为国内镜像
flatpak remote-modify flathub --url=https://mirror.sjtu.edu.cn/flathub
```

### 2. 通过flatpak按需安装以下软件

```bash
# 所见即得的markdown写作软件
flatpak install flathub org.gnome.gitlab.somas.Apostrophe
# 支持多种格式的文档浏览器
flatpak install flathub org.gnome.Papers
# epub浏览器
flatpak install flathub com.github.johnfactotum.Foliate
# 音乐播放器
flatpak install flathub com.github.neithern.g4music
```