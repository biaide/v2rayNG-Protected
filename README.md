# v2rayNG-Protected

A personal v2rayNG fork for shared Android TV and Android devices.

This fork reduces accidental configuration exposure in the user interface. It keeps normal connection, import, subscription update, testing, and deletion features, while removing common UI paths that reveal or export server details.

## Intended Use

1. Test and prepare profiles on a private device.
2. Import them into this app.
3. Use friendly profile names such as `Hong Kong`, `Japan Backup`, or `Streaming`.
4. Shared-device users can select, connect, test, update subscriptions, or delete profiles.
5. When a profile must be changed, delete it and import the replacement profile.

## Source Changes

### 1. Mask the server address on the main profile list

File:

```text
V2rayNG/app/src/main/java/com/v2ray/ang/ui/MainRecyclerAdapter.kt
```

Original code:

```kotlin
holder.itemMainBinding.tvStatistics.text = getAddress(profile)
```

Replacement:

```kotlin
holder.itemMainBinding.tvStatistics.text = "••••••"
```

Result:

```text
Hong Kong
••••••
VLESS / Reality
```

The profile remark, protocol type, latency, subscription name, and connection behavior are unchanged.

---

### 2. Hide Share, Edit, and More controls while keeping Delete

File:

```text
V2rayNG/app/src/main/java/com/v2ray/ang/ui/MainRecyclerAdapter.kt
```

Remove this original block:

```kotlin
if (doubleColumnDisplay) {
    holder.itemMainBinding.layoutRemove.visibility = View.GONE
    holder.itemMainBinding.layoutMore.visibility = View.VISIBLE
    holder.itemMainBinding.layoutMore.setOnClickListener {
        adapterListener?.onShare(guid, profile, position, true)
    }
} else {
    holder.itemMainBinding.layoutShare.visibility = View.VISIBLE
    holder.itemMainBinding.layoutEdit.visibility = View.VISIBLE
    holder.itemMainBinding.layoutRemove.visibility = View.VISIBLE
    holder.itemMainBinding.layoutMore.visibility = View.GONE

    holder.itemMainBinding.layoutShare.setOnClickListener {
        adapterListener?.onShare(guid, profile, position, false)
    }
    holder.itemMainBinding.layoutEdit.setOnClickListener {
        adapterListener?.onEdit(guid, position, profile)
    }
    holder.itemMainBinding.layoutRemove.setOnClickListener {
        adapterListener?.onRemove(guid, position)
    }
}
```

Replace it with:

```kotlin
holder.itemMainBinding.layoutShare.visibility = View.GONE
holder.itemMainBinding.layoutEdit.visibility = View.GONE
holder.itemMainBinding.layoutMore.visibility = View.GONE

holder.itemMainBinding.layoutRemove.visibility = View.VISIBLE
holder.itemMainBinding.layoutRemove.setOnClickListener {
    adapterListener?.onRemove(guid, position)
}
```

Optional cleanup: remove this unused line after removing the original `if (doubleColumnDisplay)` block:

```kotlin
private val doubleColumnDisplay =
    MmkvManager.decodeSettingsBool(AppConfig.PREF_DOUBLE_COLUMN_DISPLAY, false)
```

Result:

* Share button hidden
* Edit button hidden
* Double-column More button hidden
* QR-code and profile-share menu no longer reachable from the profile list
* Delete button remains available
* Selecting a profile remains available

---

### 3. Remove Backup & Restore from the navigation drawer

File:

```text
V2rayNG/app/src/main/res/menu/menu_drawer.xml
```

Delete this item:

```xml
<item
    android:id="@+id/backup_restore"
    android:icon="@drawable/ic_restore_24dp"
    android:title="@string/title_configuration_backup_restore" />
```

Result:

* The Backup & Restore navigation item is hidden.
* WebDAV backup and restore are no longer reachable from the normal app menu.

---

### 4. Remove Export All from the main menu

File:

```text
V2rayNG/app/src/main/res/menu/menu_main.xml
```

Delete this item:

```xml
<item
    android:id="@+id/export_all"
    android:icon="@drawable/ic_share_24dp"
    android:title="@string/title_export_all"
    app:showAsAction="never" />
```

Result:

* The Export All menu item is hidden.
* Users cannot export all profiles through the normal main-menu action.

## What This Fork Does Not Protect Against

This is a UI exposure reduction fork, not a cryptographic security product.

It does not prevent:

* Reading application data on a rooted device
* Rebuilding or patching the APK
* Runtime hooking or memory inspection
* Deleting application data
* Deleting profiles or damaging network settings

## License

This project is forked from [2dust/v2rayNG](https://github.com/2dust/v2rayNG).

The original GPL-3.0 license remains applicable.
