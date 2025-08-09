+++
date = '2025-04-18T17:06:58+08:00'
draft = true
title = 'Cannot Start Chrome Due to Singletonlock'
+++

某次打開電腦突然發現 Chromium 怎麼樣都打不開，用 terminal 執行後發現以下錯誤訊息：

```
The profile appears to be in use by another Chromium process (xxxxx)...
Chromium has locked the profile so that it doesn't get corrupted
```

(完整的 error message 可以參考[這裡](https://chromium.googlesource.com/chromium/src/+/master/chrome/app/chromium_strings.grd#1359))

上網爬了一下發現把 `.config/chromium/SingletonLock` 刪掉就可以了。

---

我們可以觀察，當我們把 `chromium` 打開的時候 `.config/chromium/` 底下會多三個檔案：

* `SingletonCookie`
* `SingletonLock`
* `SingletonSocket`

chromium source code 中的 [process_singleton_posix.cc](https://chromium.googlesource.com/chromium/src/+/HEAD/chrome/browser/process_singleton_posix.cc)

裡面講到 Chromium 會用這三個檔案來管理不同 process 間 profile 的 syncronization 

我們去反查上面出現的錯誤訊息的 ID 會發現是 `IDS_PROFILE_IN_USE_POSIX` 並且在 `DisplayProfileInUseError` 中被使用，代表下面這個 `if` 的條件是成立的

```c
if (hostname != net::GetHostName() && !IsChromeProcess(pid)) {
    // Locked by process on another host. If the user selected to unlock
    // the profile, try to continue; otherwise quit.
    if (DisplayProfileInUseError(lock_path_, hostname, pid)) {
    UnlinkPath(lock_path_);
    internal::SendRemoteProcessInteractionResultHistogram(PROFILE_UNLOCKED);
    return PROCESS_NONE;
    }
    return PROFILE_IN_USE;
}
```

推測應該是因為前一次不正常的關機使得 `SingletonLock` 沒有被正確的刪除，而且下一次開機時 PID 也已經找不到對應的 process 或是變成非 Chrome Process。

但另一個問題是為什麼 `hostname != net::GetHostName()` 會成立

剛好這台使用的電腦沒有設置 hostname (`/etc/hostname` 不存在)，查一下 hostname 的[規則](https://man.archlinux.org/man/hostname.5) 會發現 

```
a transient hostname may be set during runtime, for example based on information in a DHCP lease
```

因此這台電腦的 hostname 會根據 DHCP 拿到的 lease 而改變，可能剛好前一次拿到的 IP 跟發生問題的時候拿到的 IP 不一樣，導致 hostname 也不一樣，也剛好解釋了這次的行為。


### Reference
* https://stackoverflow.com/questions/38894321/chromium-browser-singleton-bug-caused-by-power-failure