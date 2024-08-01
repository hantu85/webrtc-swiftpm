# WebRTC binary for SwiftPM (unfinished)
This project holds binaries of WebRTC framework (iOS & macOS) as releases.

Since release M88 we publish WebRTC framework as XCFramework compiled for:
* iOS
* iOS Simulator
* macOS (Intel)
* macOS (AppleSilicon)

Currently usage of macOS universal binary (FAT) was not yet tested.

## Building XCFramework

### Download webrtc
````sh
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=$PATH:/path/to/depot_tools

fetch --nohooks webrtc_ios

git branch -r
git checkout branch-heads/BRANCH

gclient sync
````

### Generate iOS and macOS targets
````sh
gn gen ../out/macos-x64 --args='target_os="mac" target_cpu="x64" is_component_build=false is_debug=false rtc_libvpx_build_vp9=false enable_stripping=true rtc_enable_protobuf=false rtc_enable_symbol_export=true'

gn gen ../out/macos-arm64 --args='target_os="mac" target_cpu="arm64" is_component_build=false is_debug=false rtc_libvpx_build_vp9=false enable_stripping=true rtc_enable_protobuf=false rtc_enable_symbol_export=true'

gn gen ../out/ios-arm64-device --args='target_os="ios" target_cpu="arm64" is_component_build=false use_xcode_clang=true is_debug=false  ios_deployment_target="10.0" rtc_libvpx_build_vp9=false use_goma=false ios_enable_code_signing=false enable_stripping=true rtc_enable_protobuf=false enable_ios_bitcode=false treat_warnings_as_errors=false'

gn gen ../out/ios-arm64-simulator --args='target_os="ios" target_cpu="arm64" target_environment="simulator" is_component_build=false use_xcode_clang=true is_debug=false  ios_deployment_target="10.0" rtc_libvpx_build_vp9=false use_goma=false ios_enable_code_signing=false enable_stripping=true rtc_enable_protobuf=false enable_ios_bitcode=false treat_warnings_as_errors=false'

gn gen ../out/ios-x64-simulator --args='target_os="ios" target_cpu="x64" target_environment="simulator" is_component_build=false use_xcode_clang=true is_debug=true ios_deployment_target="10.0" rtc_libvpx_build_vp9=false use_goma=false ios_enable_code_signing=false enable_stripping=true rtc_enable_protobuf=false enable_ios_bitcode=false treat_warnings_as_errors=false'
````

### Generating macOS universal (FAT) binary
````sh
mkdir out/macos-universal

cp -R out/macos-x64/WebRTC.framework out/macos-universal/WebRTC.framework

lipo -create -output out/macos-universal/WebRTC.framework/Versions/A/WebRTC out/macos-x64/WebRTC.framework/WebRTC out/macos-arm64/WebRTC.framework/WebRTC
````

### Generating iOS Simulator universal (FAT) binary
````sh
mkdir out/ios-universal-simulator

cp -R out/ios-x64-simulator/WebRTC.framework out/ios-universal-simulator/WebRTC.framework

lipo -create -output out/ios-universal-simulator/WebRTC.framework/Versions/A/WebRTC out/ios-x64-simulator/WebRTC.framework/WebRTC out/ios-arm64-simulator/WebRTC.framework/WebRTC
````

### Generating XCFramework
````sh
xcodebuild -create-xcframework \
	-framework out/ios-arm64/WebRTC.framework \
	-framework out/ios-universal-simulator/WebRTC.framework \
	-framework out/macos-universal/WebRTC.framework \
	-output out/WebRTC.xcframework
````

