# Hbuildx本地打包iOS


懒得手动打包

<!--more-->

> 务必先阅读 [官方文档](https://nativesupport.dcloud.net.cn/AppDocs/usesdk/ios)



## 配置SDK

- 打包使用的SDK版本与Hbuildx的版本一一对应，需要注意版本的匹配
- 按文档中的说明修改应用名称、图片等信息
- HBuildx中的项目ID需要与`control.xml`中的id相同



## iOS打包脚本

每次导出web资源后，需要手动在xcode中导入www并执行打包上传操作，其实这一步是可以实现自动化的，脚本如下：

```bash
#!/bin/bash

work_dir="【当前目录】"
ios_sdk="【SDK目录名称】"
project_dir="${work_dir}/${ios_sdk}/HBuilder-Hello"
xcode_project="${project_dir}/【应用名称】.xcodeproj"
www_id="【HBuildx ID】"
www="${work_dir}/【HBuildx项目名称】/unpackage/resources/${www_id}"
archive_dir="${work_dir}/archive"
released_dir="${work_dir}/release"
export_options="${work_dir}/ExportOptions.plist"
workspace="${xcode_project}/project.xcworkspace"
plist="${project_dir}/HBuilder-Hello/【应用名称】-Info.plist"
scheme="【Scheme名称】"
mode="Release"

# Verify status
old_version=`/usr/libexec/Plistbuddy -c "Print CFBundleVersion" "${plist}"`
old_build_version=`/usr/libexec/Plistbuddy -c "Print CFBundleShortVersionString" "${plist}"`
provisioning_profiles=`/usr/libexec/Plistbuddy -c "Print provisioningProfiles" "${export_options}"`
echo "App current provisioning profiles: ${provisioning_profiles}"
echo "App current version ${old_version}"
echo "App current buildVersion ${old_build_version}"

# Update version
read -p "Input new version: " version
read -p "Input new buildVersion: " build_version
echo "Update app verion to ${version}, buildVersion to ${build_version}"
/usr/libexec/Plistbuddy -c "Set CFBundleVersion ${version}" "${plist}"
/usr/libexec/Plistbuddy -c "Set CFBundleShortVersionString $build_version" "${plist}"

# Update www
echo "Delete the old ${project_dir}/HBuilder-Hello/Pandora/apps/${www_id}"
rm -r "${project_dir}/HBuilder-Hello/Pandora/apps/${www_id}"
echo "Add new ${www} to ${project_dir}/HBuilder-Hello/Pandora/apps/"
cp -r ${www} "${project_dir}/HBuilder-Hello/Pandora/apps/"


# Clean the build cache
echo "Clean the build cache"
xcodebuild clean -workspace ${workspace} -scheme ${scheme} -configuration ${mode}

# # Open project
# open ${xcode_project}

# Build
echo "Building app..."
archive_file="${archive_dir}/${scheme}v${build_version}.xcarchive"
xcodebuild -quiet archive -workspace ${workspace} -scheme ${scheme} -archivePath ${archive_file}
xcodebuild -quiet -exportArchive -archivePath ${archive_file} -exportPath "${released_dir}/v${build_version}" -exportOptionsPlist "${export_options}"

# Verify app
xcrun altool --validate-app -f "${released_dir}/v${build_version}/【应用名称】.ipa" -t "iOS" --apiKey "【API Key】" --apiIssuer "【API ID】"

# Upload app
read -p "Upload the app? Press enter to continue"
xcrun altool --upload-app -f "${released_dir}/v${build_version}/【应用名称】.ipa" -t "iOS" --apiKey "【API Key】" --apiIssuer "【API ID】"
```



工作目录示例：

```bash
ingbyr@ingbyr-mbp ~/app> tree -L 1
.
├── ExportOptions.plist
├── archive
├── build_ios.sh
├── iOSSDK@2.8.13.80344_20200927
├── app
└── release
```



脚本中的【XXX】需要根据具体项目更改，其中：

- SDK目录名称：如目录中的` iOSSDK@2.8.13.80344_20200927`

- HBuildx项目名称：如目录中的 `app`

- HBuildx ID：在HBuildx中注册账户后会分发一个ID

- API Key和API ID：用于和苹果应用商店交互，在[Apple Dev](https://developer.apple.com/account/resources/authkeys/list)中的Key下申请

- ExportOptions.plist：用于配置iOS打包选项，其中 provisioningProfiles 最好使用 AdHoc模式的配置

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
  <plist version="1.0">
  <dict>
  	<key>provisioningProfiles</key>
  	<dict>
  		<key>【Bundle ID】</key>
  		<string>【Provisioning Profile Name】</string>
  	</dict>
  	<key>compileBitcode</key>
  	<false/>
  	<key>destination</key>
  	<string>export</string>
  	<key>method</key>
  	<string>ad-hoc</string>
  	<key>signingStyle</key>
  	<string>automatic</string>
  	<key>stripSwiftSymbols</key>
  	<true/>
  	<key>teamID</key>
  	<string>【Team ID】</string>
  	<key>thinning</key>
  	<string>&lt;none&gt;</string>
  </dict>
  </plist>
  ```

  


