# 🖌 egui: 一个纯 Rust 编写的易用 GUI 库

[<img alt="github" src="https://img.shields.io/badge/github-emilk/egui-8da0cb?logo=github" height="20">](https://github.com/emilk/egui)
[![Latest version](https://img.shields.io/crates/v/egui.svg)](https://crates.io/crates/egui)
[![Documentation](https://docs.rs/egui/badge.svg)](https://docs.rs/egui)
[![unsafe forbidden](https://img.shields.io/badge/unsafe-forbidden-success.svg)](https://github.com/rust-secure-code/safety-dance/)
[![Build Status](https://github.com/emilk/egui/workflows/CI/badge.svg)](https://github.com/emilk/egui/actions?workflow=CI)
[![MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/emilk/egui/blob/master/LICENSE-MIT)
[![Apache](https://img.shields.io/badge/license-Apache-blue.svg)](https://github.com/emilk/egui/blob/master/LICENSE-APACHE)
[![Discord](https://img.shields.io/discord/900275882684477440?label=egui%20discord)](https://discord.gg/JFcEma9bJq)

👉 [点此运行 Web 样例](https://www.egui.rs/#demo) 👈

egui （读作“e-gooey”） 是一个简单、快速、可移植性强的 Rust 即时模式 GUI 库。egui 可利用WASM技术运行于Web浏览器内, 也可以像普通程序一样运行在原生平台（*Native*）， 甚至 [你喜欢的游戏引擎](#integrations) 。

egui 旨在成为最易用的 Rust GUI 库，用最简单的方式创建原生或Web应用程序。

egui 可以在任何可以绘制纹理三角形（*textured triangles*）的地方使用，这意味着你可以轻松地地将它集成到你选择的游戏引擎中。

章节:

* [示例 Example](#示例)
* [快速上手](#快速上手)
* [样例 Demo](#样例)
* [目标](#目标)
* [egui 是为谁设计的？](#egui-是为谁设计的)
* [状态/特性](#状态)
* [集成](#集成)
* [为什么使用即时模式](#为什么使用即时模式)
* [FAQ](#faq)
* [其他](#其他)
* [鸣谢](#鸣谢)

（[egui原始项目地址](https://github.com/emilk/egui)）

## 示例
原手册中的代码没头没尾，极难看懂。这里给出完整的hello_world，并且解决中文字体问题。推荐使用微软编译器stable-msvc工具链，如使用gnu工具链可能遇到输入法bug，无法唤出输入法。

*cargo.toml*
```cargo
[package]
name = "hello_world"
version = "0.1.0"
authors = ["rust"]
license = "MIT OR Apache-2.0"
edition = "2021"
rust-version = "1.71.1"

[dependencies]
eframe = { package = "eframe", version= "0.22.0", features = [
    "__screenshot", # __screenshot is so we can dump a screenshot using EFRAME_SCREENSHOT_TO
] } #不要疑惑，eframe包负责调用egui，他对egui进行了重新导出
env_logger = "0.10"
```
*rust code*
``` rust
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")] // windows_subsystem 告诉编译器，程序运行时隐藏命令行窗口。

use eframe::egui;
fn main() -> Result<(), eframe::Error> {
    env_logger::init(); // Log to stderr (if you run with `RUST_LOG=debug`).
    let options = eframe::NativeOptions {
        initial_window_size: Some(egui::vec2(320.0, 340.0)), //初始化窗体size
        ..Default::default()
    };
    eframe::run_native(
        "Hello word", //应用程序名称
        options,
        Box::new(|_cc| Box::<MyApp>::new(MyApp::new(_cc))), //第三个参数为程序构建器(eframe::AppCreator类型)负责创建应用程序上下文(egui::Context)。
//_cc为&CreationContextl类型，_cc.egui_ctx字段即为Context。
//之所以强调Context的创建过程，是因为显示中文字体需要配置Context。
    )
}

struct MyApp {
    name: String,
    age: u32,
}
impl Default for MyApp {
    fn default() -> Self {
        Self {
            name: "Arthur".to_owned(),
            age: 42,
        }
    }
}
impl MyApp{
	fn new(cc: &eframe::CreationContext<'_>) -> Self {
		load_harmony_os_font(& cc.egui_ctx); //egui默认字体无法显示中文，需要加载中文字体。配置字体应该在构造函数中。
//网上部分教程将字体配置写入了update函数，update函数每一帧都会运行一次，每秒60次，因此在update函数中加载字体是错误且低效的。
        Self::default()
    }
}

impl eframe::App for MyApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading("My egui Application");
            ui.horizontal(|ui| {
                let name_label = ui.label("Your name: ");
                ui.text_edit_singleline(&mut self.name)
                    .labelled_by(name_label.id);
            });
            ui.add(egui::Slider::new(&mut self.age, 0..=120).text("age"));
            if ui.button("Click each year").clicked() {
                self.age += 1;
            }
            ui.label(format!("Hello '{}', age {}", self.name, self.age));
        });
    }
}

// 为了支持中文，我们加载部分鸿蒙字体：下载自https://developer.harmonyos.com/cn/design/resource
//将字体文件放置在src目录同级别的resources目录下
pub fn load_harmony_os_font(ctx: &egui::Context){
    let mut fonts = eframe::egui::FontDefinitions::default();
    fonts.font_data.insert("HarmonyOS_Sans".to_owned(),
                           eframe::egui::FontData::from_static(include_bytes!("../resources/HarmonyOS_Sans_Regular.ttf"))); // .ttf and .otf supported
    fonts.font_data.insert("HarmonyOS_Sans_SC".to_owned(),
                           eframe::egui::FontData::from_static(include_bytes!("../resources/HarmonyOS_Sans_SC_Regular.ttf"))); 
	fonts.font_data.insert("HarmonyOS_Sans_TC".to_owned(),
                           eframe::egui::FontData::from_static(include_bytes!("../resources/HarmonyOS_Sans_TC_Regular.ttf"))); 
    fonts.families.get_mut(&eframe::egui::FontFamily::Proportional).unwrap()
        .insert(0, "HarmonyOS_Sans_TC".to_owned());
	fonts.families.get_mut(&eframe::egui::FontFamily::Proportional).unwrap()
        .insert(0, "HarmonyOS_Sans_SC".to_owned());
    fonts.families.get_mut(&eframe::egui::FontFamily::Proportional).unwrap()
        .insert(0, "HarmonyOS_Sans".to_owned());
    ctx.set_fonts(fonts);
}
```

<img src="media/hello_world.gif">


## 快速上手

[示例目录](https://github.com/emilk/egui/blob/master/examples/)（`examples/`）中有一些简单的示例。如果你想写一个 Web App，请按照 <https://github.com/emilk/eframe_template/>的说明操作。

官方文档位于 <https://docs.rs/egui>。要获得更多灵感或示例，请查看 [egui web 样例](https://www.egui.rs/#demo) 并按照其中的链接访问源代码。

如果你想要将egui集成到现有的引擎中，请前往 [集成](#集成) 一节。

如果有疑问，请访问 [GitHub Discussions](https://github.com/emilk/egui/discussions) 或 [egui 的 discord 服务器](https://discord.gg/JFcEma9bJq)。

如果你想为egui做贡献，请阅读 [Contributing Guidelines](https://github.com/emilk/egui/blob/master/CONTRIBUTING.md).

## 样例

[点此运行 Web 样例](https://www.egui.rs/#demo) （可运行于任何支持 WASM 和 WebGL 的浏览器）。使用 [`eframe`](https://github.com/emilk/egui/tree/master/crates/eframe)。

若要在本地测试样例 App，运行 `cargo run --release -p egui_demo_app`。

原生后端是 [`egui_glow`](https://github.com/emilk/egui/tree/master/crates/egui_glow)（使用 [`glow`](https://crates.io/crates/glow))，在 Windows 和 Mac 上开箱即用，但如果要在 Linux 上使用，需要先运行：

`sudo apt-get install -y libclang-dev libgtk-3-dev libxcb-render0-dev libxcb-shape0-dev libxcb-xfixes0-dev libxkbcommon-dev libssl-dev`

在 Fedora Rawhide 上需要运行:

`dnf install clang clang-devel clang-tools-extra libxkbcommon-devel pkg-config openssl-devel libxcb-devel gtk3-devel atk fontconfig-devel`

**注意**: 这只针对样例 App —— egui 本身是完全跨平台的！

## 目标

* 最易用的 GUI 库
* 灵敏：在 debug build 中达到 60 Hz
* 友好：难以发生编码错误，不应该发生 panic
* 可移植：同样的代码可以跨平台使用
* 轻松集成到任意环境中
* 用于自定义绘制的简单 2D 图形 API（[`epaint`](https://docs.rs/epaint)）.
* 没有回调
* 纯即时模式
* 可扩展：[轻松为 egui 编写自己的 widgets](https://github.com/emilk/egui/blob/master/crates/egui_demo_lib/src/demo/toggle_switch.rs)
* 模块化：你可以使用 egui 中的一小部分，并用新的方式将它们组合起来
* 内存安全：egui 中没有`unsafe`关键字
* 最小化依赖：[`ab_glyph`](https://crates.io/crates/ab_glyph) [`ahash`](https://crates.io/crates/ahash) [`nohash-hasher`](https://crates.io/crates/nohash-hasher) [`parking_lot`](https://crates.io/crates/parking_lot)

egui *不是*框架。egui 是供调用的库，而不是供编程的环境。

**注意**: egui 还没有实现所有上述目标！egui 仍在开发中。

### 以下并非egui的目标

* 成为最强大的 GUI 库
* 原生外观界面（*looking interface*）
* 高级灵活的布局（这与即时模式根本不兼容）

## egui 是为谁设计的？

egui 旨在成为想要以最简单的方式创建 GUI 或想要在游戏引擎中添加 GUI 的人的最佳选择。


如果你不用 Rust，egui 不适合你。如果你想要一个看起来原生的 GUI，egui 不适合你。如果你想要升级时不会破坏已有项目，egui 暂时不适合你。

但如果你想用 Rust 写一些交互式的东西，需要一个简单的 GUI，egui 可能会适合你。

### egui vs Dear ImGui

egui 的明显替代方案是 [`imgui-rs`](https://github.com/Gekkio/imgui-rs)，C++ 库 [Dear ImGui](https://github.com/ocornut/imgui) 的 Rust 封装。Dear ImGui 是一个很棒的库（也是 egui 的灵感来源），它有更多特性和打磨（*polish*）不过，egui为Rust用户提供了一些好处：

* egui 是纯 Rust 编写的
* egui 可以很方便的编译为 WASM
* egui 允许你使用原生Rust字符串类（`imgui-rs` 强制你对以零结尾的字符串使用恼人的宏和包装器）
* [Writing your own widgets in egui is simple](https://github.com/emilk/egui/blob/master/crates/egui_demo_lib/src/demo/toggle_switch.rs)

egui 还尝试在一些小地方增加你的体验：

* 窗口会根据其内容自动调整大小
* 窗口会自动定位，以避免互相重叠。
* 一些微妙的动画使 egui 变得生动

综上所述:

* egui：纯 Rust、初生、激动人心、正在开发中
* Dear ImGui：特性丰富、经过良好测试、笨重的 Rust 集成

## 状态

egui 在活跃开发中。它做的不错，但缺少许多特性，接口仍在变化。新的版本会有破坏性的改变。

### 特性

* 组件：label, text button, hyperlink, checkbox, radio button, slider, draggable value, text editing, combo box, color picker
* 布局：horizontal, vertical, columns, automatic wrapping
* 文本编辑: multiline, copy/paste, undo, emoji supports
* 窗口：move, resize, name, minimize and close. 自动调整大小和定位。
* 区域：resizing, vertical scrolling, collapsing headers (sections)
* 渲染：Anti-aliased rendering of lines, circles, text and convex polygons.
* 悬浮工具提示
* 更多

<img src="media/widget_gallery.gif" width="50%">

Light Theme:

<img src="media/light_theme.png" width="50%">

## 集成

egui 易于集成到任何你使用的游戏引擎或平台
egui 自身不知道且不关心运行它的操作系统和被渲染到屏幕的方式——这是egui集成的工作

一个集成需要在每一帧都做以下事情：

* **输入**: 采集输入（鼠标、触摸、键盘、屏幕大小……）并传递给 egui
* 运行应用程序代码
* **输出**: 处理 egui 输出 （光标变化、粘贴、纹理分配（*texture allocations*）……）

* **绘制**：渲染 egui 生成的三角形网格（参考 [OpenGL example](https://github.com/emilk/egui/blob/master/crates/egui_glow/src/painter.rs)）

### 官方集成

以下是 egui 官方集成：

* [`eframe`](https://github.com/emilk/egui/tree/master/crates/eframe) 用于将相同的App编译为 web/wasm 喝 desktop/native。使用 `egui-winit` 和 `egui_glow` 或 `egui-wgpu`.
用于在本地和web上渲染带有glow的egui，以及制作本地应用程序。
* [`egui_glow`](https://github.com/emilk/egui/tree/master/crates/egui_glow) 用于在原生平台和 Web 上渲染带有 [glow](https://github.com/grovesNL/glow) 的 egui，以及制作原生App。
* [`egui-wgpu`](https://github.com/emilk/egui/tree/master/crates/egui-wgpu) 用于 [wgpu](https://crates.io/crates/wgpu) （WebGPU API）.
* [`egui-winit`](https://github.com/emilk/egui/tree/master/crates/egui-winit) 用于集成于 [winit](https://github.com/rust-windowing/winit).
* [`egui_glium`](https://github.com/emilk/egui/tree/master/crates/egui_glium) 用于渲染带有 [Glium](https://github.com/glium/glium) 的原生App。(**已弃用** —— 正在寻找新的维护者).

### 第三方集成

* [`amethyst_egui`](https://github.com/jgraef/amethyst_egui) 用于 [the Amethyst game engine](https://amethyst.rs/).
* [`bevy_egui`](https://github.com/mvlabat/bevy_egui) 用于 [the Bevy game engine](https://bevyengine.org/).
* [`egui_glfw_gl`](https://github.com/cohaereo/egui_glfw_gl) 用于 [GLFW](https://crates.io/crates/glfw).
* [`egui-glutin-gl`](https://github.com/h3r2tic/egui-glutin-gl/) 用于 [glutin](https://crates.io/crates/glutin).
* [`egui_sdl2_gl`](https://crates.io/crates/egui_sdl2_gl) 用于 [SDL2](https://crates.io/crates/sdl2).
* [`egui_sdl2_plat用于m`](https://github.com/ComLarsic/egui_sdl2_plat用于m) 用于 [SDL2](https://crates.io/crates/sdl2).
* [`egui_vulkano`](https://github.com/derivator/egui_vulkano) 用于 [Vulkano](https://github.com/vulkano-rs/vulkano).
* [`egui_winit_vulkano`](https://github.com/hakolao/egui_winit_vulkano) 用于 [Vulkano](https://github.com/vulkano-rs/vulkano).
* [`egui-macroquad`](https://github.com/optozorax/egui-macroquad) 用于 [macroquad](https://github.com/not-fl3/macroquad).
* [`egui-miniquad`](https://github.com/not-fl3/egui-miniquad) 用于 [Miniquad](https://github.com/not-fl3/miniquad).
* [`egui_speedy2d`](https://github.com/heretik31/egui_speedy2d) 用于 [Speedy2d](https://github.com/QuantumBadger/Speedy2D).
* [`egui-tetra`](https://crates.io/crates/egui-tetra) 用于 [Tetra](https://crates.io/crates/tetra), a 2D game framework.
* [`egui-winit-ash-integration`](https://github.com/MatchaChoco010/egui-winit-ash-integration) 用于 [winit](https://github.com/rust-windowing/winit) and [ash](https://github.com/MaikKlein/ash).
* [`fltk-egui`](https://crates.io/crates/fltk-egui) 用于 [fltk-rs](https://github.com/fltk-rs/fltk-rs).
* [`ggegui`](https://github.com/NemuiSen/ggegui) 用于 the [ggez](https://ggez.rs/) game framework.
* [`godot-egui`](https://github.com/setzer22/godot-egui) 用于 [godot-rust](https://github.com/godot-rust/godot-rust).
* [`nannou_egui`](https://github.com/nannou-org/nannou/tree/master/nannou_egui) 用于 [nannou](https://nannou.cc).
* [`notan_egui`](https://github.com/Nazariglez/notan/tree/main/crates/notan_egui) 用于 [notan](https://github.com/Nazariglez/notan).
* [`screen-13-egui`](https://github.com/attackgoat/screen-13/tree/master/contrib/screen-13-egui) 用于 [Screen 13](https://github.com/attackgoat/screen-13).
* [`egui_skia`](https://github.com/lucasmerlin/egui_skia) 用于 [skia](https://github.com/rust-skia/rust-skia/tree/master/skia-safe).
* [`smithay-egui`](https://github.com/Smithay/smithay-egui) 用于 [smithay](https://github.com/Smithay/smithay/).
* [`tauri-egui`](https://github.com/tauri-apps/tauri-egui) 用于 [tauri](https://github.com/tauri-apps/tauri).

没有你想要的集成？创建一个很容易！

### 编写你自己的 egui 集成

你需要采集 [`egui::RawInput`](https://docs.rs/egui/latest/egui/struct.RawInput.html) 并处理 [`egui::FullOutput`](https://docs.rs/egui/latest/egui/struct.FullOutput.html)。基本结构如下：

``` rust
let mut egui_ctx = egui::CtxRef::default();

// Game loop:
loop {
    // Gather input (mouse, touches, keyboard, screen size, etc):
    let raw_input: egui::RawInput = my_integration.gather_input();
    let full_output = egui_ctx.run(raw_input, |egui_ctx| {
        my_app.ui(egui_ctx); // add panels, windows and widgets to `egui_ctx` here
    });
    let clipped_primitives = egui_ctx.tessellate(full_output.shapes); // creates triangles to paint

    my_integration.paint(&full_output.textures_delta, clipped_primitives);

    let platform_output = full_output.platform_output;
    my_integration.set_cursor_icon(platform_output.cursor_icon);
    if !platform_output.copied_text.is_empty() {
        my_integration.set_clipboard_text(platform_output.copied_text);
    }
    // See `egui::FullOutput` and `egui::PlatformOutput` for more
}
```

关于 OpenGl 后端请参考 [the `egui_glium` painter](https://github.com/emilk/egui/blob/master/crates/egui_glium/src/painter.rs) 或 [the `egui_glow` painter](https://github.com/emilk/egui/blob/master/egui_glow/src/painter.rs).

### 调试你的集成

#### 界面无法对齐

* 关闭 backface culling.

#### 文字看起来很模糊

* 请确保你设置了适当的 `pixels_per_point` 在 egui 的参数中.
* 确保纹理取样器没有偏离半个像素. 试着使用近邻取样器进行检查.

#### 窗口太透明或太暗

* egui 使用预乘 alpha, 所以需要确保你的混合函数功能是 `(ONE, ONE_MINUS_SRC_ALPHA)`.
* 确保你的纹理取样器使用 (`GL_CLAMP_TO_EDGE`).
* egui 更倾向于使用线性色彩空间进行混合，所以:
  * 如果可以的话，请使用sRGBA感知纹理 (e.g. `GL_SRGB8_ALPHA8`).
    * 否则：请在片段着色器中使用伽玛解码。
  * 在顶点着色器中对传入的顶点颜色使用伽玛进行解码。
  * 如果可以，打开 sRGBA或线性帧缓冲器 (`GL_FRAMEBUFFER_SRGB`).
    * 否则：在再次写入颜色之前对其使用伽马编码。


## 为什么使用即时模式

`egui` 是一个 [即时模式 GUI 库](https://en.wikipedia.org/wiki/Immediate_mode_GUI)，而不是*保留模式* GUI 库。 关于它们的区别，最好的例子就是按钮：在一个保留 GUI 中，如果你创建了一个按钮，把它添加入 UI 中，并配套点击处理程序（调用）。这个按钮会被保留在用户界面中，为改变某些界面刷新，需要存储对其调用。而相对之下，在即时模式中，这个按钮会立即显示并与你进行交互，而这对于每一帧来说都可用（每秒60帧）。这意味你不需要任何 on-click 处理程序，也不需要存储对它的任何调用。 在 `egui` 中按钮是这样表示的 `if ui.button("Save file").clicked() { save(file); }`.

*译者注：关于即时模式和保留模式的区别可以看看 [微软的这篇文章](https://docs.microsoft.com/zh-cn/windows/win32/learnwin32/retained-mode-versus-immediate-mode)。*

有关即时模式的更详细描述，请参见 [in the `egui` docs](https://docs.rs/egui/latest/egui/#understanding-immediate-mode).

这两种系统各有优缺点。

简单来说，即时模式 GUI 更容易用，但没那么强大。

### 即时模式的优点
#### 可用性
即时模式的主要优点是，使得应用程序的代码更为简单。

* 不需要有任何点击处理程序和回调操作，以免耽误你的开发流程。
* 不用担心一个不知所以然的回调调用已经删除的东西。
* GUI代码可以很容易地存在于一个简单的函数中 (无需为用户界面设置对象)。
* 不用担心应用程序状态和 GUI 状态同步问题 ( GUI 显示延迟), 因为 EGUI 不存储任何状态--即时模式。

换言之，减少大量代码、复杂性和错误，你能把时间放在比写 GUI 代码更有趣的事情上。

### 即时模式的缺点

#### Layout布局
即时模式的主要缺点是布局更加困难。假设你在屏幕中央显示一个小的对话窗口。为了正确定位该窗口，GUI库必须首先知道它的尺寸。为了知道窗口的大小，GUI库必须首先布局窗口的内容。在保留模式下，实现比较容易：GUI库进行窗口布局，定位窗口，然后检查交互情况 (按钮是否进行点击)。

即时模式下，你会遇到一个悖论：为了知道窗口的大小，我们必须进行布局，但布局代码也会检查交互（按钮是否进行点击），因此在显示窗口内容之前，需要先知道窗口的位置。这意味必须在知道窗口的大小之前就决定在哪里显示窗口。

这是即时GUI的一个基本缺陷，任何试图解决这个问题的努力都会带来自身的弊端。

一种解决方法是提前存储界面尺寸并在下一帧使用。这就产生了一个帧延迟，第一帧就会偶尔出现闪烁。 `egui` 对于如窗口和网格布局，都是如此。

也可以调用两次布局代码（一次获取尺寸，第二次进行下一次交互），但是成本更高，而且实现起来也更复杂，何况某些情况下两次不够。 `egui` 不会这样做。

对于"atomic" widgets部件 (例如一个 button) `egui` 在显示之前就知道尺寸，所以`egui`中可以将按钮、标签等放在中间，而不需要任何特殊的操作。

#### CPU usage
由于即时模式的 GUI 每帧都做了完整的布局，布局代码需要快速响应。如果你有一个非常复杂的 GUI ，这可能会对 CPU 造成负担。特别在滚动区域有一个非常大的用户界面（很长的滚动页面）将会变得很慢，因为需要将每一帧都渲染出来。

如果在设计 GUI 时考虑到了这一点，为避免使用巨大的滚动区域（或是只布置在视图中显示的部分），那么对性能影响是相当小的。在大多数情况下，你可以期望`egui`每帧占用1-2ms，但`egui`仍有很大的优化空间。也可以将`egui`设置为只在有交互时（如鼠标移动时）才进行重绘。

如果你的 GUI 是高度交互的，那么与保留模式相比，即时模式可能性能更好。进入任何网页并调整浏览器窗口的大小时候，你会注意浏览器布局非常慢，同时消耗大量 CPU 。相比之下，在 "egui "中调整一个窗口的大小，仍能保持 60FPS 帧率，且没有额外的CPU成本。


#### IDs
在一些 GUI 状态中，如果希望 GUI 库保留，或是在一个即时模式库如`egui`中。那么就需要知道窗口的位置和大小，以及在某些界面中滚动了多远。这种情况下，需要为`egui`提供唯一标识符的种子（在主 UI 中是唯一的）。例如在默认情况下，`egui`使用窗口标题作为唯一的 IDs 来存储窗口位置。如果你想要两个具有相同名称的窗口（或一个具有动态名称的窗口），你必须向`egui`提供其他的 IDs 来源（由独特的整数或字符串构成）。

`egui`需要跟踪哪个部件被交互（例如哪个滑块被拖动）。`egui`也使用唯一的 IDs，但在上述情况下，IDs 是自动生成的，所以不需担心这个问题。对于两个名字相同的按钮也没有问题（拓展 [`Dear ImGui`](https://github.com/ocornut/imgui)).

总的来说，IDs 处理有些不方便，但并不是一个大的缺点。


## FAQ

Also see [GitHub Discussions](https://github.com/emilk/egui/discussions/categories/q-a).

### 我可以在 `egui` 中使用非拉丁字符吗?
是的！但你需要使用 `Context::set_fonts` 安装自己的字体（`.ttf` 或 `.otf`）。

### 我可以自定义 egui 的外观吗？
是的！你可以使用 `Context::set_style` 自定义所有东西颜色、间距、字体和大小。

这里有个例子（来自 https://github.com/AlexxxRu/TinyPomodoro）

<img src="media/pompodoro-skin.png" width="50%">

### 我该如何与 `async` 一起使用 egui？
如果你在 GUI 代码中调用 `.await`，UI 会冻结，用户体验将会很差。替代方案是，保持 GUI 线程不阻塞的情况下与并发任务通信（`async` 任务或任何其他线程）。你可以用下面的方法实现。
* Channels (e.g. [`std::sync::mpsc::channel`](https://doc.rust-lang.org/std/sync/mpsc/fn.channel.html)). Make sure to use [`try_recv`](https://doc.rust-lang.org/std/sync/mpsc/struct.Receiver.html#method.try_recv) so you don't block the gui thread!
* `Arc<Mutex<Value>>` (background thread sets a value; GUI thread reads it)
* [`poll_promise::Promise`](https://docs.rs/poll-promise) (example: [`examples/download_image/`](https://github.com/emilk/egui/blob/master/examples/download_image/))
* [`eventuals::Eventual`](https://docs.rs/eventuals/latest/eventuals/struct.Eventual.html)
* [`tokio::sync::watch::channel`](https://docs.rs/tokio/latest/tokio/sync/watch/fn.channel.html)

### 无障碍功能？比如屏幕阅读器？
egui包含了可选的[AccessKit](https://accesskit.dev/)支持，它当前在Windows和macOS上实现了原生的可访问性API。这个特性在dframe中默认启用。对于AccessKit尚未支持的平台，如Web，你可以用实验性的内置屏幕阅读器；在[the web demo](https://www.egui.rs/#demo)中你可以在“Backend”选项卡打开它。

egui最开始关于无障碍的讨论在<https://github.com/emilk/egui/issues/167>。现在，AccessKit支持已合并，为未来的无障碍功能工作提供了强大的基础，请就特定的无障碍问题打开新的issue。

### [egui](https://docs.rs/egui) 和 [eframe](https://docs.rs/egui) 和 [eframe](https://github.com/emilk/egui/tree/master/crates/eframe) 的区别是什么？

`egui` 是一个有布局和交互功能的 2D 用户界面库。
`egui` 无法知道它的运行环境，也不知道如何获取输入/输出到显示器。
这是 *集成* 或 *后端* 的任务。

在游戏引擎中使用 `egui` 是很常见的（比如 [`bevy_egui`](https://docs.rs/bevy_egui)），
但你也可以依靠 `eframe` 来单独使用 `egui`。`eframe` 有着 Web 和 Native，处理输入和渲染的集成。
`eframe` 中的 _frame_ 既代表 egui app 中的 帧（frame），又代表框架（framework）（`frame` 是个框架, `egui` 是个库）。

### 我该如何在 egui 中渲染 3D 内容？
有多种方法可以将 egui 与 3D 相结合。最简单的方法是使用一个3D库，让EGUI 位于 3D 视图的顶部。例如 [`bevy_egui`](https://github.com/mvlabat/bevy_egui) or [`three-d`](https://github.com/asny/three-d).

如果想将 3D 嵌入到 egui 视图中，有两种选择。

#### `Shape::Callback`
Examples:
* <https://github.com/emilk/egui/blob/master/examples/custom_3d_glow.rs>

`Shape::Callback` will call your code when egui gets painted, to show anything using whatever the background rendering context is. When using [`eframe`](https://github.com/emilk/egui/tree/master/crates/eframe) this will be [`glow`](https://github.com/grovesNL/glow). Other integrations will give you other rendering contexts, if they support `Shape::Callback` at all.

#### Render-to-texture
You can also render your 3D scene to a texture and display it using [`ui.image(…)`](https://docs.rs/egui/latest/egui/struct.Ui.html#method.image). You first need to convert the native texture to an [`egui::TextureId`](https://docs.rs/egui/latest/egui/enum.TextureId.html), and how to do this depends on the integration you use.

Examples:
* Using [`egui-miniquad`]( https://github.com/not-fl3/egui-miniquad): https://github.com/not-fl3/egui-miniquad/blob/master/examples/render_to_egui_image.rs
* Using [`egui_glium`](https://github.com/emilk/egui/tree/master/crates/egui_glium): <https://github.com/emilk/egui/blob/master/crates/egui_glium/examples/native_texture.rs>


## 其他

### Conventions and design choices

All coordinates are in screen space coordinates, with (0, 0) in the top left corner

All coordinates are in "points" which may consist of many physical pixels.

All colors have premultiplied alpha.

egui uses the builder pattern for construction widgets. For instance: `ui.add(Label::new("Hello").text_color(RED));` I am not a big fan of the builder pattern (it is quite verbose both in implementation and in use) but until Rust has named, default arguments it is the best we can do. To alleviate some of the verbosity there are common-case helper functions, like `ui.label("Hello");`.

Instead of using matching `begin/end` style function calls (which can be error prone) egui prefers to use `FnOnce` closures passed to a wrapping function. Lambdas are a bit ugly though, so I'd like to find a nicer solution to this. More discussion of this at <https://github.com/emilk/egui/issues/1004#issuecomment-1001650754>.

egui uses a single `RwLock` for short-time locks on each access of `Context` data. This is to leave implementation simple and transactional and allow users to run their UI logic in parallel. Instead of creating mutex guards, egui uses closures passed to a wrapping function, e.g. `ctx.input(|i| i.key_down(Key::A))`. This is to make it less likely that a user would accidentally double-lock the `Context`, which would lead to a deadlock.

### Inspiration

The one and only [Dear ImGui](https://github.com/ocornut/imgui) is a great Immediate Mode GUI for C++ which works with many backends. That library revolutionized how I think about GUI code and turned GUI programming from something I hated to do to something I now enjoy.

### Name

The name of the library and the project is "egui" and pronounced as "e-gooey". Please don't write it as "EGUI".

The library was originally called "Emigui", but was renamed to "egui" in 2020.

## 鸣谢

egui author and maintainer: Emil Ernerfeldt [(@emilk](https://github.com/emilk)).

Notable contributions by:

* [@n2](https://github.com/n2): [Mobile web input and IME support](https://github.com/emilk/egui/pull/253).
* [@optozorax](https://github.com/optozorax): [Arbitrary widget data storage](https://github.com/emilk/egui/pull/257).
* [@quadruple-output](https://github.com/quadruple-output): [Multitouch](https://github.com/emilk/egui/pull/306).
* [@EmbersArc](https://github.com/EmbersArc): [Plots](https://github.com/emilk/egui/pulls?q=+is%3Apr+author%3AEmbersArc).
* [@AsmPrgmC3](https://github.com/AsmPrgmC3): [Proper sRGBA blending for web](https://github.com/emilk/egui/pull/650).
* [@AlexApps99](https://github.com/AlexApps99): [`egui_glow`](https://github.com/emilk/egui/pull/685).
* [@mankinskin](https://github.com/mankinskin): [Context menus](https://github.com/emilk/egui/pull/543).
* [@t18b219k](https://github.com/t18b219k): [Port glow painter to web](https://github.com/emilk/egui/pull/868).
* [@danielkeller](https://github.com/danielkeller): [`Context` refactor](https://github.com/emilk/egui/pull/1050).
* [@MaximOsipenko](https://github.com/MaximOsipenko): [`Context` lock refactor](https://github.com/emilk/egui/pull/2625).
* 以及 [许多贡献者](https://github.com/emilk/egui/graphs/contributors?type=a).

egui使用[MIT许可证](LICENSE-MIT) 或 [Apache-2.0许可证](LICENSE-APACHE)。

* 三次贝塞尔曲线和二次贝塞尔曲线的平坦化算法来自 [lyon_geom](https://docs.rs/lyon_geom/latest/lyon_geom/)  
*（原文：The flattening algorithm for the cubic bezier curve and quadratic bezier curve is from* [*lyon_geom*](https://docs.rs/lyon_geom/latest/lyon_geom/) *）*

默认字体：

* `emoji-icon-font.ttf`: [Copyright (c) 2014 John Slegers](https://github.com/jslegers/emoji-icon-font) , MIT License
* `Hack-Regular.ttf`: <https://github.com/source-foundry/Hack>, [MIT Licence](https://github.com/source-foundry/Hack/blob/master/LICENSE.md)
* `NotoEmoji-Regular.ttf`: [google.com/get/noto](https://google.com/get/noto), [SIL Open Font License](https://scripts.sil.org/cms/scripts/page.php?site_id=nrsi&id=OFL)
* `Ubuntu-Light.ttf` by [Dalton Maag](http://www.daltonmaag.com/): [Ubuntu font licence](https://ubuntu.com/legal/font-licence)

---

<div align="center">
<img src="media/rerun_io_logo.png" width="50%">

egui development is sponsored by [Rerun](https://www.rerun.io/), a startup doing<br>
visualizations for computer vision and robotics.
</div>
