# tinywlの疑問点

主な実装の流れ
```txt
新しい構造体を登録する
↓
ハンドラを用意
↓
listenerに登録
↓
main()で初期化
↓
wayrand実行
```

Q1.tinywlで宣言された構造体（以下）
```c
struct tinywl_server {
	struct wl_display *wl_display;
	struct wlr_backend *backend;
	struct wlr_renderer *renderer;
	struct wlr_allocator *allocator;
	struct wlr_scene *scene;
	struct wlr_scene_output_layout *scene_layout;

	struct wlr_xdg_shell *xdg_shell;
	struct wl_listener new_xdg_toplevel;
	struct wl_listener new_xdg_popup;
	struct wl_list toplevels;

	struct wlr_cursor *cursor;
	struct wlr_xcursor_manager *cursor_mgr;
	struct wl_listener cursor_motion;
	struct wl_listener cursor_motion_absolute;
	struct wl_listener cursor_button;
	struct wl_listener cursor_axis;
	struct wl_listener cursor_frame;

	struct wlr_seat *seat;
	struct wl_listener new_input;
	struct wl_listener request_cursor;
	struct wl_listener pointer_focus_change;
	struct wl_listener request_set_selection;
	struct wl_list keyboards;
	enum tinywl_cursor_mode cursor_mode;
	struct tinywl_toplevel *grabbed_toplevel;
	double grab_x, grab_y;
	struct wlr_box grab_geobox;
	uint32_t resize_edges;

	struct wlr_output_layout *output_layout;
	struct wl_list outputs;
	struct wl_listener new_output;
};
```
これは何を行っているのか？

わかっていること
- ライブラリからを用意された構造体を使いやすいように別の構造体を用意してまとめている（OOPの継承に近い）
曖昧なこと
- これはlistenerへの登録ではないのか？あくまでlistenerのポインタを宣言しているだけなのか？

Q2.tinywlで実装されている関数について（以下）
```c
static void focus_toplevel(struct tinywl_toplevel *toplevel) {

~~~~~~

static void keyboard_handle_modifiers(struct wl_listener *listener, void *data) {
```
引数にwl_listenerがあるものと無いものがある

わかってること
- wl実行中に動作するイベント処理であること
曖昧なこと
- listener引数にはどんな意図があるのか
- ハンドラとはどちらの関数を指すのか
- 引数に渡されるまでの処理の流れは一体どうなってるのか

総括：wlサーバへの機能追加の手順

プロトコルから構造体を宣言（導火線の準備）
```c
struct wl_listener anylistener
```
必要ならば再び構造体でまとめる
```c
struct tinywl_server {
	struct wl_listener anylistener1
	struct wl_listener anylistenet2
}
```
ハンドラを実装（爆弾を用意）
```c
static void anyfunc1 (//listenerを引数に紐づける（ハンドラ関数として扱う）
	struct anylistener, //結びつけるlistenerを指定
	void data //他にも引数がほしいならば追加
){
	anyfunc2(data)//必要なら関数で処理を分ける（普通の関数として使う）
}
```
シグナルへの登録（点火装置と接続）
```c
wl_signal_add(xdg_shell -> anyevents , &anylistener)
```

ここまでの疑問点

Q.1やっぱハンドラ関数がわからない
void anyfunc( anylistener //どこからもらっているのかよくわからない
)

Q.2 listenerからくるサーバーの情報をどうやって振り分けているのか構造が掴めない


