---
title: 【一行】NativeScriptでステータスバーと<ActionBar>が被るバグの対処法・決定版
tags:
  - 備忘録
date: 2021-12-20 00:44:00
---


```diff
--- a/App_Resources/Android/src/main/res/values-v21/styles.xml
+++ b/App_Resources/Android/src/main/res/values-v21/styles.xml
@@ -23,6 +23,6 @@

     <style name="NativeScriptToolbarStyle" parent="NativeScriptToolbarStyleBase">
         <item name="android:elevation">4dp</item>
-        <item name="android:paddingTop">24dp</item>
+        <item name="android:fitsSystemWindows">true</item>
     </style>
 </resources>
```

`<ActionBar>`の上paddingがハードコードされていることが原因なので、それを自動調整してくれるオプションに置換すればいい。
