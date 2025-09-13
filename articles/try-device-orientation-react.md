---
title: "【React】ジャイロセンサによるインタラクションを実現する"
emoji: "🎮"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [DeviceOrientation,React, WebAPIs, JavaScript,TypeScript]
published: true
---
この記事では、Webサイトでスマホの傾きを検知して描画に反映する方法を紹介します。

## デモ
![デモ](https://raw.githubusercontent.com/makim0939/zenn-content/main/articles/images/try-device-orientation-react/device-orientation-demo.gif =320x)

## 実装
### 実装環境
今回はVite・React・TypeScriptで実装します。
DeviceOrientationEventは**httpsのサイトでしか使えない**ので、そのための環境を作ります。
:::details Viteでhttps環境を用意
```
npm vite create@latest
```
```
npm i @vitejs/plugin-basic-ssl
```
```js: vite.config.jsを作成
import basicSsl from '@vitejs/plugin-basic-ssl'

export default {
  plugins: [
    basicSsl()
  ]
}
```
:::
### 傾きの取得
DeviceOrientationEventリスナーの登録。
最後の引数のtrueは、[`absolute`](https://developer.mozilla.org/ja/docs/Web/API/DeviceOrientationEvent/DeviceOrientationEvent#absolute)です。
- true : 地面を下とした絶対的な角度を取得します。
- false : イベントリスナが登録された時点からの相対的な角度を取得します。

```jsx
window.addEventListener("deviceorientation", handleOrientation, true);
```
取得できるデバイスの向き`alpha`, `beta`, `gamma`は以下のページで説明されています。
https://developer.mozilla.org/ja/docs/Web/API/Device_orientation_events/Orientation_and_motion_data_explained

### iOS13以降ではユーザの許可が必要

ユーザのクリックイベントから`requestPermission`を呼び出し、許可をもらう必要があります。

```jsx
const handleClick = async () => {
  // anyとして扱わないと、DeviceOrientationEventにreqiestPermissionがないというエラーが出る。
  const DOE = DeviceOrientationEvent as any;
  const result = await DOE.requestPermission();
  if (result === "granted") {
    // 許可されたときの処理
  } else if (result === "denied") {
    // 拒否されたときの処理
  }
};

...
<button onCliclk={handleClick}>許可する</button>
```

### スマホの傾きを描画に反映する

ここからは、[デモ](#デモ)の内容を最もシンプルに実装します。

```jsx
import { useEffect, useState } from "react";

function App() {
  const [orientation, setOrientation] = useState({ alpha: 0, beta: 0, gamma: 0 });

  // 使用許可を確認
  const handlePermissionBtnClick = async () => {
    const DOE = DeviceOrientationEvent as any;
    DOE.requestPermission().then(async (val: string) => {
      if (val === "granted") {
        console.log("許可されました");
      } else {
        console.log("許可されませんでした");
      }
    });
  };

  // useEffectでイベントリスナーを登録
  useEffect(() => {
    const handleOrientation = (event: DeviceOrientationEvent) => {
      setOrientation({
        alpha: event.alpha ?? 0,
        beta: event.beta ?? 0,
        gamma: event.gamma ?? 0,
      });
    };
    window.addEventListener("deviceorientation", handleOrientation, true);
    return () => {
      window.removeEventListener("deviceorientation", handleOrientation);
    };
  }, []);

  return (
    <>
      {/* 角度に応じて文字を動かしてみる */}
      <h1
        style={{
          transform:  `translateX(${(orientation.gamma * 360) / 100}px) 
                       translateY(${((orientation.beta - 30) * 360) / 100}px)`,
        }}
      >
        Try Device Orientation
      </h1>

      <button type="button" onClick={handlePermissionBtnClick}>
        方向の取得を許可する
      </button>
    </>
  );
}

export default App;
```
:::message
iOS13以降で`DOE.requestPermission()`を呼び出すと、エラーになります。
:::

### ロジックを分離して、再利用できるようにする
ここからは、カスタムフックを作成してきれいなコードにします。
ついでに、iOS13以外ではボタンが表示されないようにしておきます。

```tsx: App.tsx
import { useDeviceOrientation } from "./hooks/useDeviceOrientation";
import useDeviceType from "./hooks/useDeviceType";
import { useDoePermission } from "./hooks/useDoePermission";

function App() {
  const deviceType = useDeviceType();
  const orientation = useDeviceOrientation();
  const { doePermission, checkDoePermission } = useDoePermission();

  return (
    <>
      {/* 角度に応じて文字を動かしてみる */}
      <h1
        style={{
          transform: `translateX(${(orientation.gamma * 360) / 100}px) 
		      translateY(${((orientation.beta - 30) * 360) / 100}px)`,
        }}
      >
        Try Device Orientation
      </h1>

      {/* iOS13以降の場合は使用許可ボタンを表示 */}
      {deviceType === "iosOver13" && (
        <button
          type="button"
          onClick={() => checkDoePermission()}
          disabled={doePermission ?? false}
        >
          {doePermission ? "方向の取得が許可されています" : "方向の取得を許可する"}
        </button>
      )}
    </>
  );
}

export default App;
```
```ts:useDeviceOrientation.tsx
/* alpha, beta, gammaを取得するためのカスタムフック */
import { useEffect, useState } from "react";

type DeviceOrientation = {
  alpha: number;
  beta: number;
  gamma: number;
};

export function useDeviceOrientation() {
  const [orientation, setOrientation] = useState<DeviceOrientation>({
    alpha: 0,
    beta: 0,
    gamma: 0,
  });

  useEffect(() => {
    const handleOrientation = (event: DeviceOrientationEvent) => {
      setOrientation({
        alpha: event.alpha ?? 0,
        beta: event.beta ?? 0,
        gamma: event.gamma ?? 0,
      });
    };

    window.addEventListener("deviceorientation", handleOrientation, true);
    return () => {
      window.removeEventListener("deviceorientation", handleOrientation);
    };
  }, []);

  return orientation;
}

```

```ts:useDoePermission.ts
/* DeviceOrientationの使用許可を取得するカスタムフック */
import { useCallback, useEffect, useState } from "react";

export function useDoePermission() {
  const [doePermission, setDoePermission] = useState(false);

  const checkDoePermission = useCallback(() => {
    const DOE = DeviceOrientationEvent as any;
    DOE.requestPermission().then(async (val: string) => {
      if (val === "granted") {
        setDoePermission(true);
      } else {
        setDoePermission(false);
      }
    });
  }, []);

  // マウント時にすでに許可されているかどうか確認
  useEffect(() => {
    if (!Object.hasOwn(DeviceOrientationEvent, "requestPermission")) return;
    if (!doePermission) return;
    checkDoePermission();
  }, [doePermission, checkDoePermission]);

  // requestPermissionが使えない場合は、nullを返す
  if (!Object.hasOwn(DeviceOrientationEvent, "requestPermission")) {
    return {
      doePermission: null,
      checkDoePermission: () => {
        console.warn("requestPermission is not supported");
      },
    };
  }
  return { doePermission, checkDoePermission };
}

```

```ts:useDeviceType.ts
/* 端末のOSを取得するためのカスタムフック */
import { useEffect, useState } from "react";

export type DeviceType = "android" | "iosUnder13" | "iosOver13" | "other";

const useDeviceType = () => {
  const [deviceType, setDeviceType] = useState<DeviceType>("other");
  useEffect(() => {
    const getDeviceType = (): "android" | "iosUnder13" | "iosOver13" | "other" => {
      const ua = navigator.userAgent;
      if (ua.indexOf("Android") > 0) return "android";
      if (ua.indexOf("iPhone") > 0) {
        if (!/iPhone OS ([1-9]_|1[0-2]_)/.test(ua)) return "iosOver13";
        else return "iosUnder13";
      }
      return "other";
    };
    const deviceType = getDeviceType();
    setDeviceType(deviceType);
  }, []);
  return deviceType;
};

export default useDeviceType;
```

うまくコンポーネント化すれば、スマホの角度によって動くUI要素を簡単に複製・配置できそうですね。