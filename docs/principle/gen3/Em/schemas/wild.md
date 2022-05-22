# 野生生成

## 基本形

```
// *エンカウント判定

Slot = getSlot()
Lv = getLv()

Nature = getNature()
// *メロメロボディが有効な場合、ここにメロメロボディ判定が入る
while (PID % 25 != Nature) {
  LID = getRand()
  HID = getRand()
  PID = LID | (HID << 16)
}

// *1
HAB = getRand()
// *2
SCD = getRand()

IVs = {
  H: HAB & 0x1F,
  A: (HAB >> 5) & 0x1F,
  B: (HAB >> 10) & 0x1F,
  C: (SCD >> 5) & 0x1F,
  D: (SCD >> 10) & 0x1F,
  S: SCD & 0x1F
}
```

## 各処理
### `getSlot()`
`getRand() % 100`で得られた`0 ~ 99`の値を、エンカウント方法ごとに設定されたテーブルに対応させて決定される。
要追記。
- 草むら・洞窟
- なみのり
- ボロのつりざお
- いいつりざお
- すごいつりざお
- いわくだき

#### 磁力・静電気
Emでのみ、先頭に磁力/静電気のポケモンを置くことで、スロット生成処理が変化する。
要追記。

### `getLv()`
スロットごとに設定された最低レベル`base`とレベル変動範囲`range`によって以下のように決定される。

```
Lv = base + (getRand() % range)
```

#### プレッシャー系
Emでのみ、先頭にプレッシャー・あとなんかあったっけを置くことで、レベル決定処理が変化する。

```
Lv = base + (getRand() % range)
if (getRand() % 2 == 0)
  Lv = base + range
if (Lv != base)
  Lv -= 1
```

- レベル決定処理の後でプレッシャー判定が入っているが、記述ミスでは無く、本当にこういう処理です。
- プレッシャーが不発した場合、予め決定していたレベルから1低いレベルになりますが、本当にこういう処理です。

### `getNature()`
`getRand() % 25`で決定される。性格の内部値は別ページ参照。

#### シンクロ
Emでのみ、先頭にシンクロのポケモンを置くことで、性格決定処理が変化する。
```
if (getRand() % 2 == 0)
  Nature = synchronizePokemonsNature
else
  Nature = getRand() % 25
```

#### サファリゾーン(ホウエン)
RSおよびEmのサファリゾーンでのみ、性格決定処理が変化する。
ポロック置き場に置かれているポロックの味`taste`に応じて性格が決定する。
```
if (getRand() % 100 < 80 && taste != None) {
  array = [0, 1, 2, ..., 24]
  for (i=0; i<25; i++) {
    for (j=i+1; j<25; j++) {
      if (getRand() % 2 == 1) {
        swap(array, i, j)
      }
    }
  }

  for (i=0; i<25; i++) {
    if (array[i].likes(taste)) {
      Nature = array[i]
      break
    }
  }
} else {
  Nature = getNature()
}
```
- ポロックの有無に関わらず判定は発生する。ポロックが置かれていない場合は、判定に通っても通常の性格決定処理に移行する。
- この処理はシンクロと両立する。判定に通らなかった場合にシンクロ処理に移行する。

### 性別決定
#### メロメロボディ
Emでのみ、先頭にメロメロボディを置くことで性格値決定処理が変化する。
`getRand() % 3`で得られる値が`0`でない場合かつ、既に決定している`Slot`のポケモンが、メロメロボディ持ちのポケモンの性別と逆の性別`Gender`になる可能性がある場合、性格値決定のループ終了条件が変更される。
```
if (!Slot.GenderType.IsFixed && getRand() % 3 != 0) {
  while (PID % 25 != Nature && getGender(Slot, PID) != Gender) {
    LID = getRand()
    HID = getRand()
    PID = LID | (HID << 16)
  }
} else {
  // 通常の処理
}
```

- 通常のループ終了条件『性格値から計算される性格が`Nature`に一致している』に加え、『性格値から計算される性別が`Gender`に一致している』を満たすまで性格値計算のループが続くようになる。
- `Slot`のポケモンが性別固定あるいは性別不明の場合はメロメロボディの判定ごとスキップされ、通常の性格値決定処理が実行される。