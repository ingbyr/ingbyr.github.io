# Hbuildx本地打包iOS


务必先阅读 [Hbuildx官方文档](https://nativesupport.dcloud.net.cn/AppDocs/usesdk/ios)

<!--more-->

## 配置SDK

- 打包使用的SDK版本与Hbuildx的版本一一对应，需要注意版本的匹配
- 按文档中的说明修改应用名称、图片等信息
- HBuildx中的项目ID需要与`control.xml`中的id相同



## iOS打包脚本

每次导出web资源后，需要手动在xcode中导入www并执行打包上传操作，其实这一步是可以实现自动化的，脚本如下：

```bash
#!/bin/bash

work_dir=`pwd`
ios_sdk="[Your SDK Dir]"
project_dir="${work_dir}/${ios_sdk}/HBuilder-Hello"
xcode_project="${project_dir}/HBuilder-Hello.xcodeproj"
www_id="[Your HBuildx ID]"
www="${work_dir}/iwop-app/unpackage/resources/${www_id}"
archive_dir="${work_dir}/archive"
released_dir="${work_dir}/release"
export_options="${work_dir}/ExportOptions.plist"
workspace="${xcode_project}/project.xcworkspace"
plist="${project_dir}/HBuilder-Hello/HBuilder-Hello-Info.plist"
scheme="HBuilder"
mode="Release"
appleApiKey="[Your Apple Api Key]"
appleApiIssuer="[Your Apple User UUID]"
pgyApiKey="[Your Pyger ID]"
pgyUserKey="[Your Pyger Key]"
pgyUrl="https://www.pgyer.com/apiv1/app/upload"

# Colorful echo
echo_green() {
    echo -e "\033[32m$1\033[0m"
}

# Verify status
old_version=`/usr/libexec/Plistbuddy -c "Print CFBundleShortVersionString" "${plist}"`
old_build_version=`/usr/libexec/Plistbuddy -c "Print CFBundleVersion" "${plist}"`
provisioning_profiles=`/usr/libexec/Plistbuddy -c "Print provisioningProfiles" "${export_options}"`
echo_green "App current provisioning profiles: ${provisioning_profiles}"
echo_green "App current version ${old_version}"
echo_green "App current buildVersion ${old_build_version}"

# Update version
read -p "Input new version: " version
read -p "Input new build verions: " build_version
if [ $version ] && [ $build_version ]; then
    /usr/libexec/Plistbuddy -c "Set CFBundleShortVersionString ${version}" "${plist}"
    /usr/libexec/Plistbuddy -c "Set CFBundleVersion $build_version" "${plist}"
else
    version=${old_version}
    build_version=${old_build_version}
fi

echo_green "Use the verion ${version}, build version ${build_version}"
app_version_info="app_v${version}_b${build_version}"
ipa_dir="${released_dir}/v${build_version}"
ipa="${ipa_dir}/HBuilder.ipa"

upadte_www_resource() {
    echo_green "Delete the old ${project_dir}/HBuilder-Hello/Pandora/apps/${www_id}"
    rm -r "${project_dir}/HBuilder-Hello/Pandora/apps/${www_id}"
    echo_green "Add new ${www} to ${project_dir}/HBuilder-Hello/Pandora/apps/"
    cp -r ${www} "${project_dir}/HBuilder-Hello/Pandora/apps/"
}

clean_xcode_build_cache() {
    echo_green "Clean the build cache"
    xcodebuild clean -workspace ${workspace} -scheme ${scheme} -configuration ${mode}
}

build_ios_installer_ipa() {
    upadte_www_resource
    clean_xcode_build_cache
    echo_green "Building app..."
    archive_file="${archive_dir}/${scheme}v${build_version}.xcarchive"
    xcodebuild -quiet archive -workspace ${workspace} -scheme ${scheme} -archivePath ${archive_file}
    xcodebuild -quiet -exportArchive -archivePath ${archive_file} -exportPath "${released_dir}/v${build_version}" -exportOptionsPlist "${export_options}"
}

open_ipa_folder() {
    read -p "Open ipa folder? Enter to continue"
    open ${ipa_dir}
}

upload_pyger() {
    read -p "Upload to he pgyer.com? Enter to continue"
    echo_green "Uploading ${ipa}"
    curl -F "file=@${ipa}" -F "uKey=${pgyUserKey}" -F "_api_key=${pgyApiKey}" ${pgyUrl}
    echo ""
    echo_green "https://www.pgyer.com/dvuF"
}

verify_ipa() {
    echo_green "Verifying the ipa..."
    xcrun altool --validate-app -f ${ipa} -t "iOS" --apiKey ${appleApiKey} --apiIssuer ${appleApiIssuer}
}

upload_apple_store() {
    read -p "Upload to apple store? Enter to continue"
    xcrun altool --upload-app -f "${released_dir}/v${build_version}/HBuilder.ipa" -t "iOS" --apiKey ${appleApiKey} --apiIssuer ${appleApiIssuer}
}

open_xcode() {
    read -p "Open the Xcode? Enter to continue"
    open ${xcode_project}
}

showTips() {
    echo -e "\033[32mPlease change push cert to development mode: manifest.json => app module => push => setting\033[0m"
}

while :
do
    echo_green "Choose the action (${app_version_info})
        uw) upadte_www_resource;;
        cx) clean_xcode_build_cache;;
        bi) build_ios_installer_ipa;;
        vi) verify_ipa;;
        oi) open_ipa_folder;;
        up) upload_pyger;;
        ua) upload_apple_store;;
        ox) open_xcode;;
        q) quit;;"

    read -p "" op
    case $op in
        uw) upadte_www_resource;;
        cx) clean_xcode_build_cache;;
        bi) build_ios_installer_ipa;;
        vi) verify_ipa;;
        up) upload_pyger;;
        ua) upload_apple_store;;
        oi) open_ipa_folder;;
        ox) open_xcode;;
        q) showTips && exit;;
    esac
done
```



工作目录示例：

```bash
ingbyr@ingbyr-mbp ~/app> tree -L 1
├── ExportOptions.plist
├── archive
├── build.sh
├── iOSSDK@2.9.3.80358_20201014
├── iwop-app
└── release
```



脚本中的【XXX】需要根据具体项目更改，其中：

- SDK Dir ： 上图中的`iOSSDK@2.9.3.80358_20201014`

- HBuildx ID：在HBuildx中注册账户后会分发一个ID

- Apple Api Key：用于和苹果应用商店交互，在[Apple Dev](https://developer.apple.com/account/resources/authkeys/list)中的Key下申请

- Apple User UUID：同上

- Pyger ID：蒲公英账户的API ID

- Pyger Key：蒲公英账户的Key

- ExportOptions.plist：用于配置iOS打包选项，其中 provisioningProfiles 最好使用 AdHoc模式的配置

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
  <plist version="1.0">
  <dict>
  	<key>provisioningProfiles</key>
  	<dict>
  		<key>[Bundle ID]</key>
  		<string>[Provisioning Profile Name]</string>
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
  	<string>[Team ID]</string>
  	<key>thinning</key>
  	<string>&lt;none&gt;</string>
  </dict>
  </plist>
  ```




## 企业分发

若开发者账户类型为企业开发者,则可以通过`In House`模式打包可以直接在Safari中发布应用.`ExportOptions.plist`参考如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>compileBitcode</key>
	<false/>
	<key>destination</key>
	<string>export</string>
	<key>manifest</key>
	<dict>
		<key>appURL</key>
		<string>[Ipa文件Url]<string>
		<key>displayImageURL</key>
		<string>[57x57分辨率logo的Url]</string>
		<key>fullSizeImageURL</key>
		<string>[512x512分辨率logo的Url]</string>
	</dict>
	<key>method</key>
	<string>enterprise</string>
	<key>provisioningProfiles</key>
	<dict>
		<key>[Bundle ID]</key>
		<string>[Name]</string>
	</dict>
	<key>signingCertificate</key>
	<string>Apple Distribution</string>
	<key>signingStyle</key>
	<string>manual</string>
	<key>stripSwiftSymbols</key>
	<true/>
	<key>teamID</key>
	<string>[Team ID]</string>
	<key>thinning</key>
	<string>&lt;none&gt;</string>
</dict>
</plist>
```

其中三个Url地址需要是Https开头,否则无法正常安装. 随后随便写一个发布页面挂在网站上即可,用户通过`Safari`访问后即可安装对应应用, Html页面参考:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <title>应用名称</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta1/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-giJF6kkoqNQ00vy+HMDP7azOuL0xtbfIcaT9wjKHr8RbDVddVHyTfAAsrekwKmP1" crossorigin="anonymous">
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta1/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-ygbV9kiqUc6oa4msXn9868pTtWMgiQaeYH7/t7LECLbyPA2x65Kgf80OJFdroafW"
        crossorigin="anonymous"></script>
</head>

<body>
    <div class="container" style="width: 50rem;">
        <div class="row ">
            <div class="col">
                <p class="text-center fs-4 fw-bold">应用名称</p>
            </div>
        </div>

        <div class="row">
            <div class="col">
                <div class="card mx-auto" style="width: 20rem;">
                    <img src="./static/apple.svg" class="card-img-top mx-auto" style="width: 12rem;">
                    <div class="card-body text-center">
                        <p class="card-text mx-auto">Please open this page via safari.</p>
                        <a href="itms-services://?action=download-manifest&url=[manifest.plist的Url]"
                        class="btn btn-primary">Download iOS</a>
                    </div>
                </div>
            </div>
            <div class="col">
                <div class="card mx-auto" style="width: 20rem;">
                    <img src="./static/android.svg" class="card-img-top mx-auto" style="width: 12rem;">
                    <div class="card-body text-center">
                        <p class="card-text">Please open this page via any browser.</p>
                        <a href="[Android Apk 的 Url]" class="btn btn-primary">Download Android</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>

</html>
```

> 最重要的是 <a href="itms-services://?action=download-manifest&url=[manifest.plist的Url] 这一行

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/hbuildx%E6%9C%AC%E5%9C%B0%E6%89%93%E5%8C%85ios/  

