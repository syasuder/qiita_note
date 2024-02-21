<!--
title:   win-raw-inを使う
tags:    Python,Rawinput
id:      5abee95c4d47d9633d95
private: true
-->
# win-raw-inを使う

## win-raw-inとは

https://github.com/holl-/win-raw-in

Pythonでraw inputを使うためのモジュールです。

readme.mdにも書いてある通り、Windowsで複数の入力デバイスを識別するために必要なものです。

https://github.com/holl-/win-raw-in?tab=readme-ov-file#related-packages
> This package was inspired by the keyboard package. Unfortunately, keyboard does not distinguish between multiple input devices on Windows.


## インストール

```bash
$ pip install win-raw-in
```

## 使ってみるとエラーになる

```python
Python 3.11.2 (tags/v3.11.2:878ead1, Feb  7 2023, 16:38:35) [MSC v.1934 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import winrawin
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "C:\Python311\Lib\site-packages\winrawin\__init__.py", line 8, in <module>
    from ._api import hook_raw_input_for_window, RawInputDevice, Mouse, Keyboard, HID, RawInputEvent, list_devices
  File "C:\Python311\Lib\site-packages\winrawin\_api.py", line 7, in <module>
    from ._usb_ids import lookup_product
  File "C:\Python311\Lib\site-packages\winrawin\_usb_ids.py", line 7, in <module>
    DATABASE = usb_ids_file.read()
               ^^^^^^^^^^^^^^^^^^^
UnicodeDecodeError: 'cp932' codec can't decode byte 0x97 in position 148213: illegal multibyte sequence
>>>
```

エラーの原因は`usb.ids`ファイルのエンコーディングが`windows-1252`（あるいは`latin-1`）だからです。

latinな環境では全く問題ありませんので、バグとは言えないです。

配布元のファイルがそうなっています。
http://www.linux-usb.org/usb-ids.html

## パッチを当てる

`win-raw-in`はgithub上でissueが無効化されているようなので、手元でパッチを当てます。
だれかfolkしてprを投げてください。

```diff_python:_usb_ids.py
- with open(os.path.join(os.path.dirname(__file__), 'usb.ids'), 'r') as usb_ids_file:
+ with open(os.path.join(os.path.dirname(__file__), 'usb.ids'), 'r', encoding='latin-1') as usb_ids_file:
    DATABASE = usb_ids_file.read()
```

## 改めて使ってみる

```python
Python 3.11.2 (tags/v3.11.2:878ead1, Feb  7 2023, 16:38:35) [MSC v.1934 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import winrawin
>>> 
>>> for device in winrawin.list_devices():
...     if isinstance(device, winrawin.Mouse):
...         print(f"{device.mouse_type} name='{device.path}'")
...     if isinstance(device, winrawin.Keyboard):
...         print(f"{device.keyboard_type} with {device.num_keys} keys name='{device.path}'")
... 
Unknown type or HID keyboard with 264 keys name='\\?\HID#VID_17EF&PID_60EE&MI_00#9&4677234&0&0000#{884b96c3-56ef-11d1-bc8c-00a0c91405dd}'
HID wheel mouse name='\\?\HID#VID_17EF&PID_60EE&MI_01&Col01#9&17978e72&0&0000#{378de44c-56ef-11d1-bc8c-00a0c91405dd}'
HID wheel mouse name='\\?\HID#VID_045E&PID_0047#8&1c946f37&0&0000#{378de44c-56ef-11d1-bc8c-00a0c91405dd}'
```

→マウスが2個認識されています。

