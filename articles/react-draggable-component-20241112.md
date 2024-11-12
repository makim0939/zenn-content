---
title: "【React】ドラックで操作可能なコンポーネントの実装"
emoji: "👆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "react"]
published: true
---
## デモ
![デモ](https://raw.githubusercontent.com/makim0939/zenn-content/refs/heads/main/articles/images/react-draggable-component-20241112/demo.gif)
## 実装
### 親コンポーネントからの呼び出し
```ts:App.tsx
const App = () => {
return 
  <Draggable ignore={['input', 'textarea']}>
      <form>省略</form> //ドラッグしたいコンポーネントやHTMLElement
  </Draggable>
}
```
### Draggableコンポーネントの実装
TailwindCssでスタイリングしています。
```tsx:Draggable.tsx
import React, { useCallback, useEffect, useState } from 'react';

const Draggable = ({
  children,
  ignoreTags,
}: {
  children: React.ReactNode;
  ignoreTags?: (keyof HTMLElementTagNameMap)[];
}) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [isDragging, setIsDragging] = useState(false);
  const [downPosition, setDownPosition] = useState({ x: 0, y: 0 });
  const [upPosition, setUpPosition] = useState({ x: 0, y: 0 });

  const handleMouseDown = (e: React.MouseEvent) => {
    if (ignoreTags) {
      const upperIgnoreTags = ignoreTags?.map((tag) => tag.toUpperCase());
      const target = e.target as HTMLElement;
      if (upperIgnoreTags.includes(target.tagName)) {
        return;
      }
    }
    if (isDragging) return;
    setIsDragging(true);
    setDownPosition({
      x: e.clientX,
      y: e.clientY,
    });
  };

  const handleMouseMove = useCallback(
    (e: React.MouseEvent | MouseEvent) => {
      if (!isDragging) return;
      const deltaX = e.clientX - downPosition.x;
      const deltaY = e.clientY - downPosition.y;
      setPosition({
        x: upPosition.x + DeltaX,
        y: upPosition.y + DeltaY,
      });
    },
    [isDragging, downPosition, upPosition],
  );
  const handleMouseUp = () => {
    setIsDragging(false);
    setUpPosition({ x: position.x, y: position.y });
  };

  useEffect(() => {
    window.addEventListener('mousemove', handleMouseMove);
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, [handleMouseMove]);

  return (
    <div className=" w-[100vw] h-[100vh] ">
      <div
        onMouseDown={handleMouseDown}
        onMouseMove={handleMouseMove}
        onMouseUp={handleMouseUp}
        style={{
          transform: `translate(${position.x}px, ${position.y}px)`,
        }}
        className=" absolute top-0 left-0 w-fit z-10 cursor-pointer origin-top-left"
      >
        {children}
      </div>
    </div>
  );
};
export default Draggable;
```
## 解説
- cssの`transform: translate()`を使って要素を動かしています。
```
動かす距離("position") = 
最後に離した位置("upPosition") 
+ 押された位置("downPosition") 
- 現在のカーソルの位置("clientX, clientY")
```
- Draggable内の特定の要素をドラッグできないようにする。
プロップ`ignore`として指定したHTMLElementのタグ名の配列を指定することで、それに当てはまる要素をドラッグできないようにしています。
```ts: handleMouseDown関数
const handleMouseDown = (e: React.MouseEvent) => {
    if (ignoreTags) {
      const upperIgnoreTags = ignoreTags?.map((tag) => tag.toUpperCase());
      const target = e.target as HTMLElement;
      if (upperIgnoreTags.includes(target.tagName)) {
        return;
      }
    }
   //startPositionの更新処理　...
```