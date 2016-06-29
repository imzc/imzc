.net core RTM 了，Azure global 已经提供支持，但是慢半拍的 Azure 中国 WebSites 目前并不支持 asp.net core。
    
    Orz
    
经过各种尝试，终于发现了临时解决的办法：
    
    让我们把整个 .net core 上传上去把
    
操作步骤
- 将系统安装的.net core 文件夹上传到 `wwwroot` 的父级目录, 让我们能够在 `wwwroot` 使用 `..\dotnet\dotnet.exe` 访问到它。
- 修改发布后的 `web.config` 将 `configuration\system.webServer\aspNetCore@processPath` 由 `%LAUNCHER_PATH%` 或 `dotnet` 改为 `..\dotnet\dotnet.exe`

    (●'◡'●)

有一个问题是，Visual Studio 发布项目后，会自动把 `processPath` 改为 `dotnet`，避免每次上传手动修改，我们在 `project.json` 中添加一个后处理脚本

```javascript
  // project.json
  "scripts": {
    "postpublish": [
      "dotnet publish-iis --publish-folder %publish:OutputPath% --framework %publish:FullTargetFramework%",
      "node azurecn.js %publish:OutputPath%" // <====== 执行 node 脚本，替换processPath
    ]
  }
  
  // azurecn.js
  var fs = require('fs');
  var path = require('path');
  var folder = process.argv[2];
  var filePath = path.join(folder, 'web.config');
  console.log(filePath);
  var content = fs.readFileSync(filePath, 'utf-8');
  if (content.indexOf('processPath="dotnet"') > 0) {
      content = content.replace('processPath="dotnet"', 'processPath="..\dotnet\dotnet.exe"');
      fs.writeFileSync(filePath, content);
  }
  
```

    整个世界都美好了！(●'◡'●)
    
    
