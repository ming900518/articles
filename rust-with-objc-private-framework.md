# 在 Rust 專案中使用 macOS 的 Objective-C Private Framework

## 前言

眾所週知， Apple 非常喜歡在自己的軟體裡塞一堆好用的功能，但不開放給用戶去用，這些功能通常會被 Apple 存放在每一台 Mac/iDevice 的 `/System/Library/PrivateFrameworks` 資料夾中，作為 dynamic library 存在

想要在 Rust 專案取用這些 Private Framework ，需要先對專案進行額外設定，下面以 `SidecarCore.framework` （macOS Sidecar 功能）為例

## 安裝 Dependency

Rust 本身是沒有類似 Swift `dlopen()` 之類用於載入 dynamic library 的功能，所以需要利用其他 crate 來實現這個功能，這邊我選擇了看起來較為活躍的 `libloading`

```
$ cargo add libloading
```

引入 dynamic library 時，需要根據其實作的語言來找相對應的 binding ， `SidecarCore.framework` 用的是 Objective-C ，所以需要引入 `objc2` 套件

```
$ cargo add objc2
```

## 找到真正的 dynamic library 路徑

Apple 在近年的 macOS 中把這些 framework 的 library 檔給隱藏起來了，像本文的範例 `SidecarCore.framework` ，如果去翻它所在的目錄（`/System/Library/PrivateFrameworks/SidecarCore.framework`），會發現根本沒有可以作為目標的檔案！

遇到這種情況不用擔心，直接將 Private Framework 的名稱貼在路徑後即可

```
/System/Library/PrivateFrameworks/SidecarCore.framework/SidecarCore
```

> 如果在後面的流程中程式提示沒辦法引入 library ，那有可能是 Apple 已經把這東西移除或改名了

## 範例 Rust Code

```rust
fn main() {
    // 引入 dynamic library
    let _sidecar_core_lib = unsafe {
        libloading::Library::new(
            "/System/Library/PrivateFrameworks/SidecarCore.framework/SidecarCore",
        ).unwrap()
    };

    // 利用 objc2 提供的 macro 載入需要的 class
    let sidecar_display_manager: &AnyClass = class!(SidecarDisplayManager);

    // 進行訊息傳遞，取得啓用功能所需的 sharedManager
    let shared_manager: *mut NSObject = unsafe { sidecar_display_manager.send_message(sel!(sharedManager), ()) };

    // 取得所有可用設備
    // 想使用 Objective-C 的 NSArray ，需要多引入 objc2-foundation crate ，並啓用 NSArray feature
    let devices: *mut NSArray = unsafe { shared_manager.send_message(sel!(devices), ());

    // 利用 objc2-foundation crate 的 NSEnumerator feature （需額外啓用），將 NSArray 轉換成 Rust 的 Vec<&AnyObject>
    let device_list = unsafe {
        devices
            .as_ref()
            .unwrap()
            .iter()
            .collect::<Vec<&AnyObject>>()
    };

    // 下面示範透過 SidecarCore 提供的功能，利用 device_list 中的第一個設備啓用 Sidecar 功能
    // 由於連線的 method 需要傳入 Objective-C 的 Block ，先行建立
    let completion_closure = StackBlock::new(|_: &AnyObject| ());

    // 將 device_list 中的第一個設備與先行建立好的 Block 傳遞到 sharedManager 中
    // 成功的話，Mac 就會連線到指定的設備並啓用 Sidecar 了
    unsafe {
        let _: bool = msg_send![shared_manager.as_ref().unwrap(), connectToDevice: device_list[0] completion: &completion_closure];
    }
}
```
