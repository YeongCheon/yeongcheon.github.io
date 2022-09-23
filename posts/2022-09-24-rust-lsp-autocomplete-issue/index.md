# Emacs LSP 환경에서 Rust 자동완성 안되는 문제 수정


* 증상
Ubuntu + Emacs(lsp-mode) 조합으로 Rust 개발환경을 셋팅하는 도중 ~wasm~ 관련 코드가 자동완성이 안되는 문제가 발생했다.
* Sample Code
** Cargo.toml

#+BEGIN_SRC rust
[package]
name = "helloWasm"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[lib]
crate-type = ["cdylib"]

[dependencies]
js-sys = "0.3.60"
wasm-bindgen = "0.2.83"

[dependencies.web-sys]
version = "0.3.60"
features = [
  'CanvasRenderingContext2d',
  'Document',
  'Element',
  'HtmlCanvasElement',
  'Window',
]


#+END_SRC

** lib.rs

#+BEGIN_SRC rust
#[wasm_bindgen]
pub fn start() {
    let window: Window = web_sys::window().unwrap();
    let document: Document = window.document().unwrap();

    let canvas = document.get_element_by_id("canvas").unwrap();

    let canvas: web_sys::HtmlCanvasElement = canvas
        .dyn_into::<web_sys::HtmlCanvasElement>()
        .map_err(|_| ())
        .unwrap();

    let context = canvas
        .get_context("2d")
        .unwrap()
        .unwrap()
        .dyn_into::<web_sys::CanvasRenderingContext2d>()
        .unwrap();

    context.begin_path();
    // Draw the outer circle.
    context
        .arc(75.0, 75.0, 50.0, 0.0, f64::consts::PI * 2.0)
        .unwrap();

    // Draw the mouth.
    context.move_to(110.0, 75.0);
    context.arc(75.0, 75.0, 35.0, 0.0, f64::consts::PI).unwrap();

    // Draw the left eye.
    context.move_to(65.0, 65.0);
    context
        .arc(60.0, 65.0, 5.0, 0.0, f64::consts::PI * 2.0)
        .unwrap();

    // Draw the right eye.
    context.move_to(95.0, 65.0);
    context
        .arc(90.0, 65.0, 5.0, 0.0, f64::consts::PI * 2.0)
        .unwrap();

    context.stroke();
}
#+END_SRC
* 해결법
해당 프로젝트 내에서 ~cargo check~ 명령어를 실행하면 다음과 같은 에러가 발생한다.

[[./error.png]]

위와 같은 에러가 발생하면 ~sudo apt install build-essential~ 명령어를 실행하면 해결할 수 있다.

