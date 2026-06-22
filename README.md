# v2rayNG-Protected

A personal v2rayNG fork for shared Android TV and Android devices.

This fork reduces accidental configuration exposure in the user interface. It keeps normal connection, import, subscription update, testing, and deletion features, while removing common UI paths that reveal or export server details.

## Intended Use

1. Test and prepare profiles on a private device.
2. Import them into this app.
3. Shared-device users can select, connect, test, update subscriptions, or delete profiles.
4. When a profile must be changed, delete it and import the replacement profile.

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

### 3. Remove Backup & Restore from the Navigation Drawer

File:

```text
V2rayNG/app/src/main/res/menu/menu_drawer.xml
```

Remove the `backup_restore` menu item.

Also remove the matching navigation handler from:

```text
V2rayNG/app/src/main/java/com/v2ray/ang/ui/MainActivity.kt
```

Remove:

```kotlin
R.id.backup_restore ->
    requestActivityLauncher.launch(Intent(this, BackupActivity::class.java))
```

Result:

* Backup & Restore is not visible in the normal navigation drawer.
* WebDAV backup and restore are not reachable through the normal app UI.
* `BackupActivity` remains in the source code but has no normal GUI entry point.

---

### 4. Remove Export All from the Main Menu

File:

```text
V2rayNG/app/src/main/res/menu/menu_main.xml
```

Remove the `export_all` menu item.

Also remove the matching main-menu handler from:

```text
V2rayNG/app/src/main/java/com/v2ray/ang/ui/MainActivity.kt
```

Remove:

```kotlin
R.id.export_all -> {
    exportAll()
    true
}
```

Result:

* Export All is not visible in the normal main menu.
* Users cannot export all profiles through the normal app UI.
* The internal `exportAll()` implementation remains in the source code but has no normal GUI entry point.

---

### UI-Only Scope

This fork intentionally reduces exposure through normal Android UI controls.

It does not remove internal import, export, backup, or edit implementations from the source code. Those functions are simply not reachable through the normal user interface after the relevant menu items and profile action controls are removed.
