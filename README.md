簡潔な記法からFDPSの相互作用計算カーネルを生成できるようにすることを考える。

とりあえず、
```
        racc -o kernelparser.rb kernelparser.y
	ruby parserdriver.rb
```

で、パースされた構文木と復元した実行文がでる。 if の実装はまだ。
定数もない。

型: とりあえずあらかじめ定義した

```
 f[64,32,16]{vec}
 i[64,32,16]
```

だけ。整数実装してない。f16もまだ
fxx と fyy の演算では語長長いほうでやる。型推論もそうなる。
fxx と iyy では fxx にする。

演算

 * スカラーとベクトルならベクトル、ベクトルとベクトルなら内積
 * + - 普通に
 * +=, -= (*= とかはなしで)
 * / 右側はスカラー
 * foo(x,y,z ...) どちらもスカラー、外部関数
 * if block なんかつける(まだ未実装)

あと定数も必要だがまだ未実装

文のきれめ

改行で必ず変わる。継続行は "\" 入れる(未実装)パーサーの作りは今は適当
にで正規表現とRipper.tokenize が混在。将来的にはもうちょっと情報残して
コンパイル時エラーが見えるようにするべき。

サンプル

```
  epj jparticle
  epi iparticle
  fi  force
  iin f64vec xi r
  jin f64vec xj r
  jin f64    mj m
  iin f64 eps2
  iout f64vec f
  iout f64 phi
  rij = xi-xj
  r2 = rij*rij+eps2
  rinv = rsqrtinv(r2)
  mrinv = mj*rinv
  phi = phi-mrinv
  f = f-mrinv*rinv*rinv* rij
```

で、 "noconverion" で

```
  f64vec rij = (xi[i]-xj[j])
  f64 r2 = ((rij*rij)+eps2[i])
  f64 rinv = rsqrtinv(r2)
  f64 mrinv = (mj[j]*rinv)
  phi[i] = (phi[i]-mrinv)
  f[i] = (f[i]-(((mrinv*rinv)*rinv)*rij))
```

と型推論とループ用インデックス付加したコードをだす。


P3T用なら

```
  epj jparticle
  epi iparticle
  fi  force
  iin f64vec xi
  iin f64    ri2
  jvar f64vec xj
  jvar f64    mj
  jvar f64    rj2
  jvar i32    j
  iout f64vec
  iout f64 phi
  iout i32 jmin
  iout i32 nj
  rij = xi-xj
  r2 = rij*rij
  rij2max = max(ri2,rj2)
  if r2 < rij2max
     r2 = rij2max
     nj += 1
     jmin = j
  end	
  rinv = rsqrtinv(r2)
  mrinv = mj*rinv
  p -= mrinv
  f -= mrinv*rinv*rinv* rij
```

これから普通のコードや他の色々なものを生成する。
