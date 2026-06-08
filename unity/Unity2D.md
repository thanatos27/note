---
tags:
  - Unity
  - CSharp
  - GameDev
  - 2D
created: 2026-06-03
updated: 2026-06-03
---

# Unity 2D リファレンス

## 目次

- [概要](#概要)
- [セットアップ](#セットアップ)
- [プロジェクト構成・バージョン管理](#プロジェクト構成バージョン管理)
- [GameObject と Component](#gameobject-と-component)
- [MonoBehaviour ライフサイクル](#monobehaviour-ライフサイクル)
- [2D シーンの基本](#2d-シーンの基本)
  - [Sprite](#sprite)
  - [Sorting Layer](#sorting-layer)
- [Tilemap](#tilemap)
- [Transform・2D 移動](#transform2d-移動)
- [入力処理](#入力処理)
  - [Input System](#input-system)
  - [旧 Input Manager](#旧-input-manager)
- [2D 物理・当たり判定](#2d-物理当たり判定)
  - [Rigidbody2D](#rigidbody2d)
  - [Collider2D と Trigger](#collider2d-と-trigger)
  - [Raycast2D](#raycast2d)
- [Camera 2D](#camera-2d)
- [アニメーション](#アニメーション)
- [UI (uGUI)](#ui-ugui)
- [Prefab・インスタンス生成](#prefabインスタンス生成)
- [ScriptableObject](#scriptableobject)
- [コルーチン・非同期](#コルーチン非同期)
- [シーン管理・ゲーム管理](#シーン管理ゲーム管理)
- [Audio](#audio)
- [セーブデータ](#セーブデータ)
- [デバッグ・プロファイリング](#デバッグプロファイリング)
- [ビルド・Player Settings](#ビルドplayer-settings)
- [よく使う API](#よく使う-api)
- [Godot との比較](#godot-との比較)
- [Tips](#tips)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## 概要

Unity は 2D / 3D ゲームに対応したゲームエンジン。スクリプトは **C#** を標準的に使う。  
このメモは **Unity 6.0 / 6.3 LTS** を前提に、主に **2D ゲーム制作** で使う機能をまとめる。

Unity は商用エンジンで、利用条件はプランと収益・資金調達額によって変わる。Runtime Fee は撤廃済みなので、最新の条件は公式の Pricing / Terms を確認する。

---

## セットアップ

**Unity Hub** からエディタをインストールし、2D テンプレートでプロジェクトを作成する。

- Editor Version は LTS を選ぶ
- 2D プロジェクトでは URP 2D Renderer を使うと 2D Light などを扱いやすい
- Package Manager で必要に応じて Input System、Cinemachine、Addressables などを追加する

---

## プロジェクト構成・バージョン管理

整理例:

```
MyGame/
├── Assets/
│   ├── Animations/
│   ├── Art/
│   ├── Audio/
│   ├── Prefabs/
│   ├── Scenes/
│   ├── Scripts/
│   ├── ScriptableObjects/
│   └── Tilemaps/
├── Packages/
│   └── manifest.json
├── ProjectSettings/
└── Library/
```

Git に含める:

- `Assets/`
- `Packages/manifest.json`, `Packages/packages-lock.json`
- `ProjectSettings/`
- すべての `.meta` ファイル

Git から除外する:

- `Library/`
- `Temp/`
- `Obj/`
- `Build/`, `Builds/`
- `Logs/`
- `UserSettings/`

画像・音声・動画などの大きいバイナリは Git LFS の利用を検討する。`.meta` ファイルを消すと GUID が変わり、Prefab や Scene の参照が壊れるので必ず管理対象にする。

---

## GameObject と Component

Unity の基本単位は **GameObject** と **Component**。

- GameObject は入れ物
- Component を付けることで Sprite 表示、物理、音声、スクリプトなどの機能を持つ
- すべての GameObject は `Transform` を持つ
- 2D オブジェクトでも Transform は `Vector3`。基本は `x`, `y` を使い、`z` は描画順や奥行き調整に使う

```csharp
// 同じ GameObject に付いている Rigidbody2D を取得する
Rigidbody2D rb = GetComponent<Rigidbody2D>();

// 子オブジェクトを含めて SpriteRenderer を探す
SpriteRenderer sprite = GetComponentInChildren<SpriteRenderer>();

// 実行時に Component を追加する
gameObject.AddComponent<AudioSource>();

// GameObject を無効化する。Update や当たり判定も止まる
gameObject.SetActive(false);
```

### Inspector への公開

```csharp
public class Player : MonoBehaviour
{
    // public フィールドは Inspector に表示され、他のスクリプトからも変更できる
    public float speed = 5f;

    // private のまま Inspector に表示する。外部から直接変更されたくない値に使う
    [SerializeField] private int maxHp = 100;

    // Inspector から参照を割り当てる。GetComponent の呼び忘れや取得コストを避けられる
    [SerializeField] private Rigidbody2D rb;

    // public だが Inspector には表示しない
    [HideInInspector] public int score;
}
```

`public` は外部コードからも変更できる。Inspector にだけ出したい値は `private` + `[SerializeField]` が扱いやすい。

---

## MonoBehaviour ライフサイクル

```csharp
public class Example : MonoBehaviour
{
    void Awake()
    {
        // 自分自身の参照取得・初期化
    }

    void OnEnable()
    {
        // 有効化時。イベント購読など
    }

    void Start()
    {
        // 最初のフレーム更新前。他オブジェクト参照の初期化など
    }

    void FixedUpdate()
    {
        // 固定時間間隔。Rigidbody2D を使う物理処理
    }

    void Update()
    {
        // 毎フレーム。入力、タイマー、非物理の更新
    }

    void LateUpdate()
    {
        // Update 後。カメラ追従など
    }

    void OnDisable()
    {
        // 無効化時。イベント購読解除など
    }

    void OnDestroy()
    {
        // 破棄時。後始末
    }
}
```

簡略イメージ:

```
初期化: Awake → OnEnable → Start
物理:   FixedUpdate は固定タイムステップごとに 0 回以上
更新:   Update → LateUpdate が描画フレームごと
終了:   OnDisable / OnDestroy は無効化・破棄時
```

`FixedUpdate` は毎フレーム必ず `Update` の後に呼ばれるわけではない。物理フレームの都合で、1 フレーム内に複数回呼ばれたり、呼ばれないこともある。

---

## 2D シーンの基本

### Sprite

`SpriteRenderer` で画像を表示する。下記の例では、Animator を使わずに Sprite の差し替えと左右反転だけで見た目を更新している。

```csharp
using UnityEngine;

[RequireComponent(typeof(SpriteRenderer))]
public class PlayerView : MonoBehaviour
{
    [SerializeField] private Sprite idleSprite;
    [SerializeField] private Sprite runSprite;

    private SpriteRenderer spriteRenderer;

    void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
    }

    public void UpdateView(Vector2 velocity, bool damaged)
    {
        bool isMoving = Mathf.Abs(velocity.x) > 0.01f;

        // 止まっているときと移動中で表示する Sprite を切り替える
        spriteRenderer.sprite = isMoving ? runSprite : idleSprite;

        // 左に移動しているときだけ左右反転する
        if (velocity.x != 0f)
            spriteRenderer.flipX = velocity.x < 0f;

        // ダメージ中だけ赤くする。通常時は白に戻す
        spriteRenderer.color = damaged ? Color.red : Color.white;
    }
}
```

画像インポートでよく見る設定:

- Texture Type: `Sprite (2D and UI)`
- Sprite Mode: 単体なら `Single`、スプライトシートなら `Multiple`
- Pixels Per Unit: 1 ユニットあたりのピクセル数
- Filter Mode: ドット絵なら `Point`
- Compression: ドット絵や UI は圧縮でにじまないよう注意

### Sorting Layer

2D の描画順は主に以下で制御する。

- `Sorting Layer`
- `Order in Layer`
- 必要に応じて Transform の `z`

背景、地形、キャラクター、エフェクト、UI などで Sorting Layer を分けると管理しやすい。

---

## Tilemap

Tilemap は、タイル画像をマス目に塗ってステージを作る仕組み。床、壁、背景、装飾などを大量の GameObject として置くのではなく、`Grid` と `Tilemap` でまとめて管理できる。

主な要素:

| 要素 | 役割 |
|------|------|
| Sprite | タイルに使う元画像。スプライトシートなら Sprite Editor で分割する |
| Tile Asset | Sprite をタイルとして塗れるようにしたアセット |
| Tile Palette | Tile Asset を並べた作業用パレット。ここからブラシで Scene に塗る |
| Grid | Tilemap の親。セルの大きさや並び方を管理する |
| Tilemap | 実際にタイルが配置されるレイヤー |
| Tilemap Renderer | Tilemap を描画する Component |
| Tilemap Collider 2D | Tilemap に 2D 当たり判定を付ける Component |

基本の作成手順:

1. タイル画像を `Sprite (2D and UI)` としてインポートする
2. スプライトシートの場合は Sprite Editor で `Multiple` に分割する
3. **Window → 2D → Tile Palette** を開く
4. Tile Palette を作成し、Sprite をドラッグして Tile Asset を生成する
5. Hierarchy で **2D Object → Tilemap → Rectangular** を作成する
6. Tile Palette からブラシを選び、Scene ビュー上の Tilemap に塗る

レイヤーを分ける例:

```
Grid
├── BackgroundTilemap   # 背景。Collider なし
├── GroundTilemap       # 地形。Collider あり
└── DecorationTilemap   # 草・看板など。Collider なし
```

当たり判定を付ける場合は、地形用 Tilemap に `Tilemap Collider 2D` を追加する。タイルごとの細かい Collider が多くなる場合は `Composite Collider 2D` を組み合わせると、隣接したタイルの当たり判定をまとめられる。

Composite Collider 2D を使うときの構成:

- `Tilemap Collider 2D` の `Used By Composite` を有効にする
- 同じ GameObject に `Rigidbody2D` を追加する
- `Rigidbody2D` の Body Type を `Static` にする
- `Composite Collider 2D` を追加する

Tilemap は描画順も重要。背景、地形、前景を別 Tilemap に分け、`Tilemap Renderer` の Sorting Layer / Order in Layer で順番を調整する。

---

## Transform・2D 移動

```csharp
Vector3 pos = transform.position;
transform.position = new Vector3(1f, 2f, 0f);

// 相対移動
transform.Translate(Vector2.right * speed * Time.deltaTime);

// 2D 回転。Z 軸を回す
transform.rotation = Quaternion.Euler(0f, 0f, 90f);

// 目的地へ移動
transform.position = Vector3.MoveTowards(
    transform.position,
    targetPosition,
    speed * Time.deltaTime
);
```

物理オブジェクトを動かす場合は、Transform を直接書き換えるより `Rigidbody2D` を使う方が衝突判定と相性がよい。

---

## 入力処理

### Input System

新規プロジェクトでは **Input System** を使うのが基本。Package Manager で追加したあと、`.inputactions` アセットを作成して入力アクションを定義する。

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

[RequireComponent(typeof(Rigidbody2D))]
public class PlayerController2D : MonoBehaviour
{
    [SerializeField] private float speed = 6f;
    [SerializeField] private float jumpVelocity = 12f;
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundRadius = 0.1f;
    [SerializeField] private LayerMask groundMask;

    private PlayerInputActions actions;
    private Rigidbody2D rb;
    private Vector2 moveInput;
    private bool jumpRequested;

    void Awake()
    {
        actions = new PlayerInputActions();
        rb = GetComponent<Rigidbody2D>();
    }

    void OnEnable()
    {
        actions.Player.Enable();
        actions.Player.Jump.performed += OnJump;
    }

    void OnDisable()
    {
        actions.Player.Jump.performed -= OnJump;
        actions.Player.Disable();
    }

    void Update()
    {
        // Move は毎フレーム読み、FixedUpdate で物理に反映する
        moveInput = actions.Player.Move.ReadValue<Vector2>();
    }

    void FixedUpdate()
    {
        rb.linearVelocity = new Vector2(moveInput.x * speed, rb.linearVelocity.y);

        if (jumpRequested && IsGrounded())
            rb.linearVelocity = new Vector2(rb.linearVelocity.x, jumpVelocity);

        // 1回だけ反応させる入力なので、物理処理後にリセットする
        jumpRequested = false;
    }

    private void OnJump(InputAction.CallbackContext context)
    {
        // performed は押した瞬間に呼ばれる。実際のジャンプ処理は FixedUpdate で行う
        jumpRequested = true;
    }

    private bool IsGrounded()
    {
        return Physics2D.OverlapCircle(groundCheck.position, groundRadius, groundMask);
    }
}
```

Action 名の例:

| Action | Value Type | 用途 |
|--------|------------|------|
| `Move` | `Vector2` | 移動 |
| `Jump` | `Button` | ジャンプ |
| `Attack` | `Button` | 攻撃 |
| `Pause` | `Button` | ポーズ |

### 旧 Input Manager

古い API だが、短い試作ではまだ使われる。

```csharp
void Update()
{
    float h = Input.GetAxisRaw("Horizontal");

    if (Input.GetKeyDown(KeyCode.Space))
        Jump();
}
```

プロジェクト設定の Active Input Handling によっては旧 API が効かないことがある。新規プロジェクトでは Input System を優先する。

---

## 2D 物理・当たり判定

### Rigidbody2D

2D 物理には `Rigidbody2D` を使う。3D 用の `Rigidbody` / `Collider` とは別物。

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody2D))]
public class Bullet2D : MonoBehaviour
{
    [SerializeField] private float speed = 14f;
    [SerializeField] private float lifeTime = 3f;

    private Rigidbody2D rb;

    void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    void OnEnable()
    {
        // 一定時間後に消す。プール運用なら Destroy ではなく SetActive(false) にする
        Destroy(gameObject, lifeTime);
    }

    public void Launch(Vector2 direction)
    {
        // 弾は重力の影響を受けない設定にして、指定方向へ一定速度で飛ばす
        rb.gravityScale = 0f;
        rb.linearVelocity = direction.normalized * speed;
    }

    void OnCollisionEnter2D(Collision2D collision)
    {
        // 壁や敵に当たったら消す
        Destroy(gameObject);
    }
}
```

よく使う設定:

| 設定 | 用途 |
|------|------|
| Body Type: Dynamic | プレイヤー、敵、弾など動く物体 |
| Body Type: Static | 地面、壁など動かない物体 |
| Gravity Scale | 重力の強さ |
| Freeze Rotation Z | キャラクターが倒れないようにする |
| Collision Detection | 高速移動のすり抜け対策 |

### Collider2D と Trigger

```csharp
void OnCollisionEnter2D(Collision2D collision)
{
    if (collision.gameObject.CompareTag("Enemy"))
        TakeDamage();
}

void OnTriggerEnter2D(Collider2D other)
{
    if (other.CompareTag("Item"))
        PickUp(other.gameObject);
}
```

- 物理的にぶつけたい場合は `Collider2D`
- 重なりだけ検知したい場合は `Is Trigger`
- レイヤー同士の衝突は **Project Settings → Physics 2D** の Layer Collision Matrix で制御する

### Raycast2D

`Raycast2D` は、指定した位置から指定方向へ見えない線を飛ばし、最初に当たった `Collider2D` を調べる。床までの距離確認、壁の検知、視線判定、クリック位置の判定などに使う。

```csharp
int groundMask = LayerMask.GetMask("Ground");

// 現在位置から下方向へ 1.2 ユニットだけ Ray を飛ばし、Ground レイヤーだけを調べる
RaycastHit2D hit = Physics2D.Raycast(transform.position, Vector2.down, 1.2f, groundMask);

if (hit.collider != null)
{
    Debug.Log(hit.point);      // 当たった位置
    Debug.Log(hit.normal);     // 当たった面の向き
    Debug.Log(hit.collider);   // 当たった Collider2D
}
```

足元の接地判定のように「この範囲に地面があるか」を見るだけなら、`OverlapCircle` も使いやすい。Raycast は方向と距離を見るのに向いていて、Overlap 系は範囲内の有無を見るのに向いている。

```csharp
[SerializeField] private Transform groundCheck;
[SerializeField] private float groundRadius = 0.1f;
[SerializeField] private LayerMask groundMask;

bool IsGrounded()
{
    return Physics2D.OverlapCircle(groundCheck.position, groundRadius, groundMask);
}
```

---

## Camera 2D

基本は `Camera` を Orthographic にする。Orthographic は遠近感のない投影方式で、カメラから遠いものも近いものも同じ大きさで表示される。2D ゲームでは絵のサイズをそのまま見せたいことが多いので、Perspective より Orthographic が向いている。

- Projection: `Orthographic`
- Size: 表示する縦方向の広さ。値を大きくすると広い範囲が映り、キャラクターは小さく見える
- Background: 背景色
- Culling Mask: 映すレイヤー

追従カメラは Cinemachine を使うと楽。

- `Cinemachine Camera` を作成
- Follow に Player を指定
- Dead Zone / Damping で追従感を調整
- Pixel art では Pixel Perfect Camera も検討する

---

## アニメーション

Unity のアニメーションは、主に **Animation Clip**、**Animator Controller**、**Animator Component** の 3 つで動く。

| 要素 | 役割 |
|------|------|
| Animation Clip | `Idle`, `Run`, `Attack` など、実際の動きやスプライト差し替えを保存した素材 |
| Animator Controller | Clip を状態として並べ、どの条件で `Idle → Run` のように遷移するかを管理する |
| Animator Component | GameObject に付ける Component。Animator Controller を参照して、実行時にアニメーションを再生する |

流れとしては、Sprite を並べて Animation Clip を作り、その Clip を Animator Controller の State に配置し、キャラクターの Animator Component に Controller を割り当てる。コードからは Animator Parameter を変更して、Controller 側の遷移条件を動かす。

```csharp
using UnityEngine;

public class PlayerAnimation : MonoBehaviour
{
    [SerializeField] private Animator animator;
    [SerializeField] private Rigidbody2D rb;
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundRadius = 0.1f;
    [SerializeField] private LayerMask groundMask;

    void Update()
    {
        // Animator Controller 側に同名の Float Parameter「Speed」を作っておく
        animator.SetFloat("Speed", Mathf.Abs(rb.linearVelocity.x));

        // Bool Parameter「Grounded」を遷移条件に使う
        animator.SetBool("Grounded", IsGrounded());

        // 例: 攻撃ボタンが押されたら Attack State へ遷移させる
        if (Input.GetKeyDown(KeyCode.Z))
            PlayAttack();
    }

    public void PlayAttack()
    {
        // Trigger Parameter「Attack」は攻撃アニメーションのような一度きりの遷移に使う
        animator.SetTrigger("Attack");
    }

    private bool IsGrounded()
    {
        return Physics2D.OverlapCircle(groundCheck.position, groundRadius, groundMask);
    }
}
```

よく使う Animator Parameter:

| Parameter | 用途 |
|-----------|------|
| `Float` | 速度、向き、ブレンド値 |
| `Bool` | 接地中、攻撃中などの状態 |
| `Trigger` | ジャンプ、攻撃、被弾など一度きりの遷移 |
| `Int` | 状態番号、武器種別 |

Sprite の左右反転は、単純な横スクロールなら `SpriteRenderer.flipX` が簡単。子オブジェクトの当たり判定や攻撃判定も反転させたい場合は、親の `localScale.x` を反転する方法もある。

---

## UI (uGUI)

ゲーム中の HUD やメニューは Canvas の下に配置する。ランタイム UI では uGUI が現在も広く使われる。

`Canvas` は UI を描画するためのルート。`Button`, `Image`, `Slider`, `TextMeshProUGUI` などの UI オブジェクトは、基本的に Canvas の子として置く。Canvas の下にある UI は `Transform` ではなく `RectTransform` で位置、サイズ、アンカーを管理する。

Canvas の主な Render Mode:

| Render Mode | 用途 |
|-------------|------|
| Screen Space - Overlay | 画面に直接 UI を重ねる。HUD やメニューで一番使いやすい |
| Screen Space - Camera | 指定した Camera を通して UI を描画する。カメラ演出や描画順を調整したい場合 |
| World Space | UI をワールド内のオブジェクトとして置く。吹き出し、看板、敵のHPバーなど |

解像度対応には `Canvas Scaler` を使う。基本は `UI Scale Mode` を `Scale With Screen Size` にし、基準解像度を決めておく。

```csharp
using TMPro;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.UI;

public class HUD : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI scoreText;
    [SerializeField] private Slider hpBar;
    [SerializeField] private Button retryButton;

    void Start()
    {
        retryButton.onClick.AddListener(OnRetryClicked);
    }

    void OnDestroy()
    {
        retryButton.onClick.RemoveListener(OnRetryClicked);
    }

    public void SetScore(int score)
    {
        scoreText.text = $"Score: {score}";
    }

    public void SetHp(float ratio)
    {
        hpBar.value = ratio;
    }

    private void OnRetryClicked()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }
}
```

- テキストは Legacy Text より **TextMeshPro (TMP)** を使う
- 位置と伸縮は `RectTransform` の Anchor / Pivot で決める
- `Canvas Scaler` の基準解像度はプロジェクトの想定画面に合わせる
- ワールド上に出す UI は World Space Canvas か Sprite ベースの表示を検討する

---

## Prefab・インスタンス生成

Prefab は、GameObject の構成をアセットとして保存したもの。敵、弾、アイテム、エフェクト、UI パーツのように何度も使うものを Prefab にしておくと、Scene に配置したり、コードから生成したりできる。

`Instantiate` は Prefab から実体を作る処理。生成した GameObject は Scene 上に存在し、`Destroy` で破棄できる。

```csharp
[SerializeField] private GameObject bulletPrefab;
[SerializeField] private Transform firePoint;

void Fire()
{
    GameObject bullet = Instantiate(bulletPrefab, firePoint.position, firePoint.rotation);
    Destroy(bullet, 3f);
}
```

頻繁に生成・破棄する弾やエフェクトは Object Pool を使う。Object Pool は、使い終わったオブジェクトを破棄せず無効化して取っておき、次に必要になったとき再利用する仕組み。`Instantiate` / `Destroy` の回数を減らせるので、負荷や GC によるカクつきを抑えやすい。

```csharp
using UnityEngine;
using UnityEngine.Pool;

public class BulletPool : MonoBehaviour
{
    [SerializeField] private GameObject bulletPrefab;
    private ObjectPool<GameObject> pool;

    void Awake()
    {
        pool = new ObjectPool<GameObject>(
            createFunc: () => Instantiate(bulletPrefab),
            actionOnGet: obj => obj.SetActive(true),
            actionOnRelease: obj => obj.SetActive(false),
            actionOnDestroy: obj => Destroy(obj),
            maxSize: 50
        );
    }

    public GameObject GetBullet()
    {
        return pool.Get();
    }

    public void ReleaseBullet(GameObject bullet)
    {
        pool.Release(bullet);
    }
}
```

---

## ScriptableObject

設定値やゲームデータをアセットとして管理するクラス。

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "ItemData", menuName = "Game/Item Data")]
public class ItemData : ScriptableObject
{
    public string itemName;
    public Sprite icon;
    public int value;
}
```

```csharp
public class Item : MonoBehaviour
{
    [SerializeField] private ItemData data;

    void Start()
    {
        Debug.Log(data.itemName);
    }
}
```

- Project ウィンドウで右クリック → **Create → Game → Item Data**
- シーンに置かず、アセットとして保存される
- 複数のオブジェクトから同じデータを参照できる
- ランタイム中に値を書き換えると共有データへの変更になるので注意

---

## コルーチン・非同期

### コルーチン

コルーチンは、処理の途中で一時停止して、次のフレームや一定時間後に続きを実行できる仕組み。点滅、演出待ち、クールダウン、数秒後に消す処理など、「少し待ってから続けたい」場面で使う。

通常のメソッドは呼び出したら最後まで一気に実行されるが、コルーチンは `yield return` で途中停止できる。

```csharp
void Start()
{
    StartCoroutine(Flash());
}

IEnumerator Flash()
{
    spriteRenderer.color = Color.red;
    yield return new WaitForSeconds(0.1f);
    spriteRenderer.color = Color.white;
}
```

`yield return null` は次フレームまで待つ。`WaitForSeconds` は `Time.timeScale` の影響を受ける。

### async / await

標準の `Task` / `async` も使えるが、Unity API は基本的にメインスレッドから触る。フレーム待機、キャンセル、PlayerLoop 連携を多用する場合は **UniTask** が便利。

```csharp
using Cysharp.Threading.Tasks;
using UnityEngine;

public class Loader : MonoBehaviour
{
    public async UniTask LoadAsync()
    {
        // 例: ローディング演出を 1 秒だけ見せる
        await UniTask.Delay(1000);

        // Unity の PlayerLoop 上で次の Update タイミングまで待つ
        await UniTask.Yield(PlayerLoopTiming.Update);

        // ここで実際のロード完了後の処理を行う
        Debug.Log("Loaded");
    }
}
```

`UniTaskVoid` はイベントハンドラや fire-and-forget 用。呼び出し側で待てる処理は、基本的に `UniTask` を返す。

---

## シーン管理・ゲーム管理

シーン切り替えには `SceneManager` を使う。タイトル、ゲーム本編、リザルトなどを別 Scene に分け、必要なタイミングで読み込む。

ゲーム全体で 1 つだけ存在してほしい管理オブジェクトには Singleton を使うことがある。下の例では `GameManager.Instance` からどこでも同じ GameManager を参照できるようにし、シーンをまたいでも消えないようにしている。

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    // static にすることで GameManager.Instance から参照できる
    public static GameManager Instance { get; private set; }

    // private set にして、外部から勝手に書き換えられないようにする
    public int Score { get; private set; }

    void Awake()
    {
        // すでに GameManager が存在するなら、重複して作られた自分を消す
        if (Instance != null)
        {
            Destroy(gameObject);
            return;
        }

        // 最初に作られた GameManager を共有インスタンスとして登録する
        Instance = this;

        // シーンを切り替えても、この GameObject を破棄しない
        DontDestroyOnLoad(gameObject);
    }

    public void LoadGame()
    {
        // GameScene に切り替える
        SceneManager.LoadScene("GameScene");
    }

    public void Retry()
    {
        // 現在のシーンを読み直す
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }
}
```

```csharp
// 非同期ロード。ロード画面を出しながら次のシーンを準備したいときに使う
AsyncOperation op = SceneManager.LoadSceneAsync("GameScene");

// true にするまでシーン遷移を完了させない
op.allowSceneActivation = false;

// Unity の仕様として、allowSceneActivation=false の間は op.progress が 0.9 で止まる。
// ローディング演出が終わったら allowSceneActivation=true にして、シーン切り替えを完了させる。
```

グローバル管理は便利だが、すべてを Singleton にすると依存が追いにくくなる。スコア、シーン遷移、設定など用途を絞る。

---

## Audio

```csharp
using UnityEngine;

[RequireComponent(typeof(AudioSource))]
public class PlayerSfx : MonoBehaviour
{
    [SerializeField] private AudioClip jumpClip;

    private AudioSource audioSource;

    void Awake()
    {
        audioSource = GetComponent<AudioSource>();
    }

    public void PlayJump()
    {
        // PlayOneShot は現在再生中の音を止めずに、効果音を重ねて鳴らせる
        audioSource.PlayOneShot(jumpClip);
    }
}
```

- `AudioSource` は音を鳴らす Component
- `AudioClip` は音声アセット
- BGM は `loop` を有効にした AudioSource で再生
- 音量カテゴリ管理には `AudioMixer` を使う
- 2D サウンドは Spatial Blend を 0 にする

---

## セーブデータ

簡単な設定や進行状況は JSON + `Application.persistentDataPath` に保存する。

`Application.persistentDataPath` は、Unity が用意しているユーザーごとの保存先パス。OS やプラットフォームごとに実際の場所は変わるが、アプリを終了しても残るデータの保存先として使える。

```csharp
using System;
using System.IO;
using UnityEngine;

[Serializable]
public class SaveData
{
    public int highScore;
    public int coins;
}

public static class SaveSystem
{
    // 式形式プロパティ。呼び出されるたびに保存先パスを組み立てる
    private static string SavePath => Path.Combine(Application.persistentDataPath, "save.json");

    public static void Save(SaveData data)
    {
        // SaveData を JSON 文字列に変換する。true は読みやすい整形表示
        string json = JsonUtility.ToJson(data, true);

        // ファイルに書き込む。既に存在する場合は上書きされる
        File.WriteAllText(SavePath, json);
    }

    public static SaveData Load()
    {
        // セーブファイルがまだない場合は、初期状態のデータを返す
        if (!File.Exists(SavePath))
            return new SaveData();

        // JSON ファイルを読み込み、SaveData に戻す
        string json = File.ReadAllText(SavePath);
        return JsonUtility.FromJson<SaveData>(json);
    }
}
```

`PlayerPrefs` は、Unity が用意している簡易的な設定保存機能。`int`, `float`, `string` をキー名で保存できるので、音量、解像度、キー設定、最後に選んだステージなどの軽い設定に向いている。

Windows ではレジストリに保存される。複雑なセーブデータや一覧データには、JSON などのファイル保存を使う。

---

## デバッグ・プロファイリング

```csharp
// 通常のログ。値の確認や処理が通ったかを見る
Debug.Log("message");

// 警告ログ。動くが設定ミスの可能性がある場合など
Debug.LogWarning("warning");

// エラーログ。処理を続けられない問題や想定外の状態を出す
Debug.LogError("error");

// Scene ビューに赤い Ray を 1 秒間描く。Raycast の向きや距離確認に使う
Debug.DrawRay(transform.position, Vector2.down, Color.red, 1f);
```

Gizmos は Scene ビューに常時表示する補助線や範囲表示に使う。攻撃範囲、視界、接地判定などを確認しやすい。

```csharp
using UnityEngine;

public class GroundCheckGizmo : MonoBehaviour
{
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundRadius = 0.1f;

    void OnDrawGizmosSelected()
    {
        if (groundCheck == null)
            return;

        // オブジェクト選択中だけ、接地判定の範囲を黄色い円で表示する
        Gizmos.color = Color.yellow;
        Gizmos.DrawWireSphere(groundCheck.position, groundRadius);
    }
}
```

よく使う確認:

- Console でエラーとスタックトレースを見る
- Scene ビューで Collider / Gizmos を確認する
- **Physics 2D Debugger** で当たり判定を確認する
- Profiler で CPU、GC Alloc、Rendering を見る
- Frame Debugger で描画順やドローコールを確認する

---

## ビルド・Player Settings

ビルド前に確認する項目:

- 対象シーンが Build Settings / Build Profiles に入っているか
- Company Name / Product Name
- 解像度、Fullscreen Mode
- アイコン、スプラッシュ、カーソル
- Scripting Backend
- 対象プラットフォームの SDK

主な出力先:

- Windows / macOS / Linux
- Web
- Android / iOS

Web やモバイルは入力、解像度、ファイル保存パス、音声再生タイミングの挙動が変わりやすいので、早めに実機・実ブラウザで確認する。

---

## よく使う API

```csharp
// 時間
Time.deltaTime;
Time.fixedDeltaTime;
Time.time;
Time.timeScale;

// GameObject
gameObject.SetActive(false);
Destroy(gameObject);
Instantiate(prefab, position, rotation);

// Component
GetComponent<Rigidbody2D>();
TryGetComponent(out Collider2D collider2d);

// 検索。多用しない
GameObject.Find("Player");
GameObject.FindGameObjectWithTag("Player");

// 数学
Mathf.Clamp(value, min, max);
Mathf.Lerp(a, b, t);
Mathf.Abs(x);
Mathf.Sign(x);

// ランダム
float r = Random.Range(0f, 1f);
int n = Random.Range(0, 10);

// Vector2
Vector2.Distance(a, b);
Vector2.Dot(a, b);
a.normalized;
a.magnitude;
```

---

## Godot との比較

| 項目 | Unity 2D | Godot 2D |
|------|----------|----------|
| スクリプト | C# が標準 | GDScript / C# |
| 基本単位 | GameObject + Component | Node |
| 初期化 | `Awake()` / `Start()` | `_ready()` |
| 毎フレーム | `Update()` | `_process(delta)` |
| 物理フレーム | `FixedUpdate()` | `_physics_process(delta)` |
| 入力 | Input System / Input Manager | Input Map |
| 2D 表示 | SpriteRenderer | Sprite2D |
| 2D 物理 | Rigidbody2D / Collider2D | CharacterBody2D / RigidBody2D / Area2D |
| Tilemap | Grid + Tilemap | TileMapLayer |
| イベント | C# event / UnityEvent | Signal |
| Prefab 相当 | Prefab | Scene |
| データアセット | ScriptableObject | Resource |
| シーン管理 | SceneManager | SceneTree |
| ライセンス | プラン条件あり | MIT |

---

## Tips

- **2D では 2D 用 Component を使う** — `Rigidbody2D` と `Rigidbody` は別物
- **物理操作は FixedUpdate に寄せる** — 入力は Update、物理反映は FixedUpdate に分けると安定しやすい
- **`Time.deltaTime` を使う** — Transform 移動はフレームレート非依存にする
- **`GetComponent` はキャッシュする** — 毎フレームの取得を避ける
- **タグ比較は `CompareTag` を使う**
- **Layer と Sorting Layer を分けて考える** — 衝突制御は Layer、描画順は Sorting Layer
- **Prefab は小さく保つ** — Player、Bullet、Enemy、Item など再利用単位で分ける
- **ScriptableObject はマスターデータ向け** — アイテム、敵パラメータ、ステージ設定に便利
- **`Find` 系は控えめに** — Inspector 参照、生成時注入、Manager 経由を優先する
- **`.meta` を必ずコミットする** — 参照切れ防止に重要

---

## 用語集

| 用語 | 説明 |
|------|------|
| GameObject | シーン上の基本オブジェクト |
| Component | GameObject に機能を追加する部品 |
| Transform | 位置・回転・スケール |
| Sprite | 2D 画像として扱うアセット |
| SpriteRenderer | Sprite をシーンに描画する Component |
| Rigidbody2D | 2D 物理で動くための Component |
| Collider2D | 2D 当たり判定 |
| Trigger | 物理衝突せず重なりだけ検知する Collider |
| Tilemap | タイルを並べてステージを作る仕組み |
| Sorting Layer | 2D の描画順を分けるレイヤー |
| Prefab | 再利用できる GameObject のテンプレート |
| ScriptableObject | データをアセットとして保存するクラス |
| Scene | 画面・ステージ・メニューなどの単位 |
| Canvas | UI を配置するルート |
| TextMeshPro | 高品質なテキスト描画システム |
| Animator | Animation Clip の遷移を管理する Component |
| Input System | Unity の新しい入力管理パッケージ |
| Addressables | アセット読み込み・配信を管理する仕組み |
| Application.persistentDataPath | セーブデータ保存に使うユーザー固有パス |

---

## リンク集

### 公式

- [Unity 公式サイト](https://unity.com/)
- [Unity 6 Releases](https://unity.com/releases/unity-6)
- [Unity Manual](https://docs.unity3d.com/Manual/)
- [Unity Scripting API](https://docs.unity3d.com/ScriptReference/)
- [2D game development](https://docs.unity3d.com/Manual/Unity2D.html)
- [Input System](https://docs.unity3d.com/Packages/com.unity.inputsystem@latest)
- [Tilemap](https://docs.unity3d.com/Manual/class-Tilemap.html)
- [Unity Pricing](https://unity.com/products/pricing-updates)

### 学習・周辺

- [Unity Learn](https://learn.unity.com/)
- [Unity Discussions](https://discussions.unity.com/)
- [Unity Asset Store](https://assetstore.unity.com/)
- [Cysharp UniTask](https://github.com/Cysharp/UniTask)
