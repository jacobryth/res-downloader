## Mac
```bash
wails build -platform "darwin/universal"
create-dmg 'build/bin/res-downloader.app' --overwrite ./build/bin
mv -f "build/bin/res-downloader $(jq -r '.info.productVersion' wails.json).dmg" "build/bin/res-downloader_$(jq -r '.info.productVersion' wails.json)_mac.dmg"
```

## Windows
```bash
wails build -f -nsis -platform "windows/amd64" -webview2 Embed -skipbindings && mv -f "build/bin/res-downloader-amd64-installer.exe" "build/bin/res-downloader_$(jq -r '.info.productVersion' wails.json)_win_amd64.exe"
wails build -f -nsis -platform "windows/arm64" -webview2 Embed -skipbindings && mv -f "build/bin/res-downloader-arm64-installer.exe" "build/bin/res-downloader_$(jq -r '.info.productVersion' wails.json)_win_arm64.exe"
```

## Linux

###  docker方式
> x86_64
```bash
docker build --network host -f build/linux/dockerfile -t res-downloader-amd-linux .
docker run -it --name res-downloader-amd-build --network host --privileged -v ./:/www/res-downloader res-downloader-amd-linux /bin/bash
# 容器内
cd /www/res-downloader
wails build -platform "linux/amd64" -s -skipbindings

# 打包debian
cp build/bin/res-downloader build/linux/Debian/usr/local/bin/
echo "$(cat build/linux/Debian/DEBIAN/.control | sed -e "s/{{Version}}/$(jq -r '.info.productVersion' wails.json)/g" -e "s/{{Architecture}}/amd64/g")" > build/linux/Debian/DEBIAN/control
dpkg-deb --build ./build/linux/Debian build/bin/res-downloader_$(jq -r '.info.productVersion' wails.json)_linux_amd64.deb

# 打包AppImage
cp build/bin/res-downloader build/linux/AppImage/usr/bin/

# 复制WebKit相关文件
pushd build/linux/AppImage

for f in WebKitNetworkProcess WebKitWebProcess libwebkit2gtkinjectedbundle.so; do
    path=$(find /usr/lib* -name "$f" 2>/dev/null | head -n 1)
    if [ -n "$path" ]; then
        mkdir -p ./$(dirname "$path")
        cp --parents "$path" .
    else
        echo "⚠️ $f not found, you may need to install libwebkit2gtk"
    fi
done

popd

# 下载appimagetool
# Note: if the GitHub download is slow, you can use a mirror or pre-download the tool manually
# Mirror alternative: https://mirror.ghproxy.com/https://github.com/AppImage/AppImageKit/releases/download/13/appimagetool-x86_64.AppImage
wget -O ./build/bin/appimagetool-x86_64.AppImage https://github.com/AppImage/AppImageKit/releases/download/13/appimagetool-x86_64.AppImage 
chmod +x ./build/bin/appimagetool-x86_64.AppImage
./build/bin/appimagetool-x86_64.AppImage build/linux/AppImage build/bin/res-downloader_$(jq -r '.info.productVersion' wails.json)_linux_amd64.AppImage

mv -f build/bin/res-downloader build/bin/res-downloader_$(jq -r '.info.productVersion' wails.json)_linux_amd64
```

> arm64
```bash
# arm
docker build --platform linux/arm64 --network host -f build/linux/dockerfile -t res-downloader-arm-linux .
docker run --platform linux/arm64 -it --name res-downloader-arm-build --network host --privileged -v ./:/www/res-downloader res-downloader-arm-linux /bin/bash
# 容器内
cd /www/res-downloader
wails build -platform "linux/arm64" -s -skipbindings

# 打包debian
cp build/bin/res-downloader build/linux/Debian/usr/local/bin/
echo "$(cat build/linux/Debian/DEBIAN/.control | sed -e "s/{{V
```
