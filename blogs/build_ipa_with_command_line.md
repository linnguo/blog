##使用命令行打包ipa的方法
###一种方式是先编译成app，然后使用xrun打包成ipa

	xcodebuild build -target Unity-iPhone 
		-scheme Unity-iPhone 
		-derivedDataPath ~/Desktop/ 
		-configuration Release

	xcrun -sdk iphoneos 
		PackageApplication -v ~/Desktop/Build/Products/throne.app 
		-o ~/Desktop/throne.ipa

编译的时候可以指定Provisioning和CodeSign，Provision的参数是UUID，Provision安装完之后，会放在~/Library/MobileDevice/Provisioning Profiles目录下，以UUID为文件名；实测后发现，指定Provisioning后，CodeSign指定没有效果，会按照Provision推导出来；

	xcodebuild build -target Unity-iPhone 
		-scheme Unity-iPhone 
		-derivedDataPath ~/Desktop/ 
		-configuration Debug
		PROVISIONING_PROFILE=36516e39-5229-4c6e-9c98-120e863c9304
		CODE_SIGN_IDENTITY="iPhone Developer: XXXXXX(XXXXX)"

###另一种方式是先编译成xcarchive，然后打包成ipa
麻烦的地方是编译时指定的是Provision的UUID，打包的时候指定的是Name；好处是，xcrun的使用说明怎么也没找到，用xcodebuild更明白一些；

	xcodebuild -scheme Unity-iPhone 
		archive -archivePath ~/Desktop/ 
		-configuration Debug
		PROVISIONING_PROFILE=36516e39-5229-4c6e-9c98-120e863c9304

	xcodebuild -exportArchive 
		-exportFormat ipa 
		-archivePath ~/Desktop/throne.xcarchive/ 
		-exportPath ~/Desktop/throne.ipa 
		-exportProvisioningProfile "fwdevelopmentprofile"

###从provisioning profile中提取UUID和Name的命令如下

	$ xmllint <(security cms -D -i your.mobileprovision) --xpath '/plist/dict/key[text()="UUID"]/following-sibling::string[position()=1]/text()'
	$ xmllint <(security cms -D -i your.mobileprovision) --xpath '/plist/dict/key[text()=“Name"]/following-sibling::string[position()=1]/text()'

- 编译前，应该执行xcodebuild clean，清除中间结果
- 打包前，要先删掉旧的ipa文件，否则会出错
- provisioning profile位置 ~/Library/MobileDevice/Provisioning Profiles