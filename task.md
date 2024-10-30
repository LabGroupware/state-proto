# Task

## protobuf

リント
``` sh
buf lint
```

フォーマット修正
``` sh
buf format -w
```

破壊的変更検知
``` sh
buf breaking proto --against '.git#subdir=proto'
```

dependency解決
``` sh
buf dep update
```

build
``` sh
buf build
```

認証
``` sh
buf registry login
```

push
``` sh
buf push --label v1.0.1
```