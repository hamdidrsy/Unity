# 6. Ders - Input Sistemi ve Karakter Kontrolü

## Giriş

Oyunlarda oyuncu etkileşimi her şeydir. Klavye, mouse, gamepad veya dokunmatik ekran - hangi cihazı kullanırsanız kullanın, Unity bu girişleri yakalayıp oyun içi aksiyonlara dönüştürmenizi sağlar.

Bu derste:
- Eski Input Manager sistemi
- Yeni Input System paketi
- Karakter hareket yöntemleri
- Kamera kontrolü
- Pratik örnekler

konularını işleyeceğiz.

---

## Input Manager (Eski Sistem)

Unity'nin varsayılan input sistemidir. Basit projeler için hâlâ kullanışlıdır.

### Input Manager Ayarları

```
Edit > Project Settings > Input Manager
```

Varsayılan eksenler (Axes):
- **Horizontal**: A/D veya Sol/Sağ ok tuşları
- **Vertical**: W/S veya Yukarı/Aşağı ok tuşları
- **Fire1**: Ctrl veya Mouse 0 (sol tık)
- **Fire2**: Alt veya Mouse 1 (sağ tık)
- **Jump**: Space

---

## Klavye Girişleri

### GetKey, GetKeyDown, GetKeyUp Farkları

```csharp
void Update()
{
    // GetKey - Tuş basılı olduğu SÜRECE true döner
    if (Input.GetKey(KeyCode.W))
    {
        Debug.Log("W tuşu basılı tutuluyor");
        // Sürekli hareket için kullanılır
    }

    // GetKeyDown - Tuşa BASILDIĞI AN bir kez true döner
    if (Input.GetKeyDown(KeyCode.Space))
    {
        Debug.Log("Space tuşuna basıldı!");
        // Zıplama, ateş etme gibi tek seferlik aksiyonlar
    }

    // GetKeyUp - Tuş BIRAKILDIĞI AN bir kez true döner
    if (Input.GetKeyUp(KeyCode.Space))
    {
        Debug.Log("Space tuşu bırakıldı!");
        // Şarjlı saldırı bırakma gibi durumlar
    }
}
```

### Yaygın KeyCode'lar

```csharp
// Harfler
KeyCode.A, KeyCode.B, ... KeyCode.Z

// Sayılar
KeyCode.Alpha0, KeyCode.Alpha1, ... KeyCode.Alpha9  // Üst sıra
KeyCode.Keypad0, KeyCode.Keypad1, ...               // NumPad

// Özel tuşlar
KeyCode.Space           // Boşluk
KeyCode.Return          // Enter
KeyCode.Escape          // ESC
KeyCode.Tab             // Tab
KeyCode.LeftShift       // Sol Shift
KeyCode.RightShift      // Sağ Shift
KeyCode.LeftControl     // Sol Ctrl
KeyCode.LeftAlt         // Sol Alt

// Ok tuşları
KeyCode.UpArrow
KeyCode.DownArrow
KeyCode.LeftArrow
KeyCode.RightArrow

// Fonksiyon tuşları
KeyCode.F1, KeyCode.F2, ... KeyCode.F12
```

### Pratik Örnek: Çoklu Tuş Kontrolü

```csharp
void Update()
{
    // Shift + W = Koşma
    if (Input.GetKey(KeyCode.LeftShift) && Input.GetKey(KeyCode.W))
    {
        Debug.Log("Koşuyor!");
    }

    // Ctrl + S = Kaydetme
    if (Input.GetKey(KeyCode.LeftControl) && Input.GetKeyDown(KeyCode.S))
    {
        Debug.Log("Oyun kaydedildi!");
    }
}
```

---

## Input.GetAxis Kullanımı

GetAxis, -1 ile 1 arasında yumuşak değerler döndürür. Analog kontrol için idealdir.

### Temel Kullanım

```csharp
void Update()
{
    // Horizontal: A/D veya Sol/Sağ ok
    float horizontal = Input.GetAxis("Horizontal");
    // A veya Sol = -1, D veya Sağ = +1, hiçbiri = 0

    // Vertical: W/S veya Yukarı/Aşağı ok
    float vertical = Input.GetAxis("Vertical");
    // S veya Aşağı = -1, W veya Yukarı = +1

    Debug.Log($"H: {horizontal}, V: {vertical}");
}
```

### GetAxis vs GetAxisRaw

```csharp
void Update()
{
    // GetAxis - Yumuşak geçiş (smoothing var)
    float smoothMove = Input.GetAxis("Horizontal");
    // 0 → 0.1 → 0.3 → 0.5 → 0.7 → 0.9 → 1.0 (kademeli artar)

    // GetAxisRaw - Ani geçiş (smoothing yok)
    float rawMove = Input.GetAxisRaw("Horizontal");
    // 0 → 1 (anında değişir)
}
```

### Ne Zaman Hangisi?

| Durum | Tercih |
|-------|--------|
| Karakter hareketi (3D) | GetAxis (yumuşak) |
| 2D Platform oyunu | GetAxisRaw (keskin) |
| Araç kontrolü | GetAxis (yumuşak) |
| Menü navigasyonu | GetAxisRaw (keskin) |

---

## Mouse Girişleri

### Mouse Pozisyonu

```csharp
void Update()
{
    // Ekrandaki mouse pozisyonu (piksel cinsinden)
    Vector3 mousePos = Input.mousePosition;
    Debug.Log($"Mouse: {mousePos.x}, {mousePos.y}");

    // Mouse pozisyonunu dünya koordinatına çevir
    Vector3 worldPos = Camera.main.ScreenToWorldPoint(mousePos);
}
```

### Mouse Butonları

```csharp
void Update()
{
    // Sol tık (0)
    if (Input.GetMouseButtonDown(0))
    {
        Debug.Log("Sol tık!");
    }

    // Sağ tık (1)
    if (Input.GetMouseButtonDown(1))
    {
        Debug.Log("Sağ tık!");
    }

    // Orta tık / Tekerlek tık (2)
    if (Input.GetMouseButtonDown(2))
    {
        Debug.Log("Orta tık!");
    }

    // Basılı tutma kontrolü
    if (Input.GetMouseButton(0))
    {
        Debug.Log("Sol tuş basılı tutuluyor");
    }
}
```

### Mouse Hareketi (Delta)

```csharp
void Update()
{
    // Mouse'un bir önceki frame'den bu yana ne kadar hareket ettiği
    float mouseX = Input.GetAxis("Mouse X");
    float mouseY = Input.GetAxis("Mouse Y");

    // Kamera döndürme için kullanılır
    transform.Rotate(Vector3.up * mouseX * sensitivity);
}
```

### Mouse Tekerleği (Scroll)

```csharp
void Update()
{
    float scroll = Input.GetAxis("Mouse ScrollWheel");

    if (scroll > 0f)
    {
        Debug.Log("Yukarı scroll");
        // Zoom in veya sonraki silah
    }
    else if (scroll < 0f)
    {
        Debug.Log("Aşağı scroll");
        // Zoom out veya önceki silah
    }
}
```

---

## GetButton Sistemi

Input Manager'da tanımlı butonları kullanır. Farklı cihazları desteklemek için idealdir.

```csharp
void Update()
{
    // "Jump" butonu - Space veya Gamepad A
    if (Input.GetButtonDown("Jump"))
    {
        Jump();
    }

    // "Fire1" butonu - Ctrl veya Sol tık
    if (Input.GetButton("Fire1"))
    {
        Shoot();
    }

    // "Fire2" butonu - Alt veya Sağ tık
    if (Input.GetButtonDown("Fire2"))
    {
        Aim();
    }
}
```

### Özel Buton Tanımlama

```
Edit > Project Settings > Input Manager > Axes
```

1. Size değerini artır
2. Yeni eksen için isim ver (örn: "Sprint")
3. Positive Button: left shift
4. Kodda kullan: `Input.GetButton("Sprint")`

---

## Yeni Input System

Unity'nin modern input sistemi. Daha esnek, daha güçlü ama kurulumu biraz daha karmaşık.

### Kurulum

```
Window > Package Manager > Unity Registry > Input System > Install
```

Kurulumdan sonra Unity yeniden başlatılacak.

### Project Settings

```
Edit > Project Settings > Player > Active Input Handling
```

Seçenekler:
- **Input Manager (Old)**: Sadece eski sistem
- **Input System Package (New)**: Sadece yeni sistem
- **Both**: Her ikisi de aktif (önerilen geçiş için)

### Input Actions Asset Oluşturma

```
Project Panel > Sağ Tık > Create > Input Actions
```

### Input Actions Yapısı

```
┌─────────────────────────────────────────────────────────────┐
│ Input Actions Asset                                          │
├─────────────────────────────────────────────────────────────┤
│ ├── Action Map: "Player"                                     │
│ │   ├── Action: "Move"      → WASD, Left Stick              │
│ │   ├── Action: "Jump"      → Space, South Button           │
│ │   ├── Action: "Fire"      → Mouse Left, Right Trigger     │
│ │   └── Action: "Look"      → Mouse Delta, Right Stick      │
│ │                                                            │
│ └── Action Map: "UI"                                         │
│     ├── Action: "Navigate"  → Arrow Keys, D-Pad             │
│     ├── Action: "Submit"    → Enter, South Button           │
│     └── Action: "Cancel"    → Escape, East Button           │
└─────────────────────────────────────────────────────────────┘
```

### Action Types

```csharp
// Value - Sürekli değer (hareket, bakış)
// Her frame değer okur
// Örnek: Joystick pozisyonu, mouse delta

// Button - Basıldı/Bırakıldı
// started, performed, canceled event'leri
// Örnek: Zıplama, ateş etme

// Pass Through - Ham veri
// Tüm cihazlardan gelen veriyi geçirir
// Çoklu oyuncu veya özel durumlar için
```

### Yeni Input System Kullanımı

#### Yöntem 1: Player Input Component

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    private Vector2 moveInput;

    // Player Input component'i otomatik çağırır
    // Fonksiyon adı: On + ActionAdı
    public void OnMove(InputValue value)
    {
        moveInput = value.Get<Vector2>();
    }

    public void OnJump(InputValue value)
    {
        if (value.isPressed)
        {
            Jump();
        }
    }

    public void OnFire(InputValue value)
    {
        Fire();
    }

    void FixedUpdate()
    {
        Vector3 movement = new Vector3(moveInput.x, 0, moveInput.y);
        // Hareketi uygula
    }
}
```

#### Yöntem 2: Direct Reference

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    [SerializeField] private InputActionAsset inputActions;

    private InputAction moveAction;
    private InputAction jumpAction;

    void Awake()
    {
        var playerMap = inputActions.FindActionMap("Player");
        moveAction = playerMap.FindAction("Move");
        jumpAction = playerMap.FindAction("Jump");
    }

    void OnEnable()
    {
        moveAction.Enable();
        jumpAction.Enable();

        jumpAction.performed += OnJump;
    }

    void OnDisable()
    {
        moveAction.Disable();
        jumpAction.Disable();

        jumpAction.performed -= OnJump;
    }

    void Update()
    {
        Vector2 move = moveAction.ReadValue<Vector2>();
        // Hareketi uygula
    }

    void OnJump(InputAction.CallbackContext context)
    {
        Debug.Log("Zıpladı!");
    }
}
```

#### Yöntem 3: Generate C# Class

Input Actions asset'inde "Generate C# Class" seçeneğini işaretleyin.

```csharp
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    private PlayerInputActions inputActions; // Otomatik oluşturulan sınıf

    void Awake()
    {
        inputActions = new PlayerInputActions();
    }

    void OnEnable()
    {
        inputActions.Player.Enable();
        inputActions.Player.Jump.performed += ctx => Jump();
    }

    void OnDisable()
    {
        inputActions.Player.Disable();
    }

    void Update()
    {
        Vector2 move = inputActions.Player.Move.ReadValue<Vector2>();
        // Hareketi uygula
    }

    void Jump()
    {
        Debug.Log("Zıpladı!");
    }
}
```

---

## Karakter Hareketi

### Yöntem 1: Transform.Translate

En basit yöntem. Fizik yok, duvarlardan geçer.

```csharp
public class SimpleMovement : MonoBehaviour
{
    [SerializeField] private float speed = 5f;

    void Update()
    {
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");

        Vector3 movement = new Vector3(h, 0, v) * speed * Time.deltaTime;

        // Dünya koordinatlarına göre
        transform.Translate(movement, Space.World);

        // Nesnenin kendi yönüne göre
        // transform.Translate(movement, Space.Self);
    }
}
```

### Yöntem 2: Rigidbody

Fizik tabanlı hareket. Çarpışmalar otomatik işlenir.

```csharp
[RequireComponent(typeof(Rigidbody))]
public class RigidbodyMovement : MonoBehaviour
{
    [SerializeField] private float speed = 5f;
    [SerializeField] private float jumpForce = 5f;

    private Rigidbody rb;
    private bool isGrounded;

    void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    void Update()
    {
        // Input kontrolü Update'de
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
        }
    }

    void FixedUpdate()
    {
        // Fizik hareketi FixedUpdate'de
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");

        Vector3 movement = new Vector3(h, 0, v) * speed;

        // Yöntem 1: Velocity değiştir (Y velocity'yi koru)
        rb.velocity = new Vector3(movement.x, rb.velocity.y, movement.z);

        // Yöntem 2: MovePosition kullan
        // rb.MovePosition(transform.position + movement * Time.fixedDeltaTime);

        // Yöntem 3: AddForce kullan (kaygan his)
        // rb.AddForce(movement);
    }

    void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag("Ground"))
        {
            isGrounded = true;
        }
    }

    void OnCollisionExit(Collision collision)
    {
        if (collision.gameObject.CompareTag("Ground"))
        {
            isGrounded = false;
        }
    }
}
```

### Yöntem 3: CharacterController

Unity'nin özel karakter hareket componenti. Fizik ve transform arası bir çözüm.

```csharp
[RequireComponent(typeof(CharacterController))]
public class CharacterMovement : MonoBehaviour
{
    [Header("Hareket")]
    [SerializeField] private float walkSpeed = 5f;
    [SerializeField] private float runSpeed = 8f;
    [SerializeField] private float jumpHeight = 2f;

    [Header("Yerçekimi")]
    [SerializeField] private float gravity = -19.62f;

    private CharacterController controller;
    private Vector3 velocity;
    private bool isGrounded;

    void Awake()
    {
        controller = GetComponent<CharacterController>();
    }

    void Update()
    {
        // Yerde mi kontrolü
        isGrounded = controller.isGrounded;

        if (isGrounded && velocity.y < 0)
        {
            velocity.y = -2f; // Küçük bir aşağı kuvvet
        }

        // Hareket inputu
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");

        Vector3 move = transform.right * h + transform.forward * v;

        // Koşma kontrolü
        float currentSpeed = Input.GetKey(KeyCode.LeftShift) ? runSpeed : walkSpeed;

        controller.Move(move * currentSpeed * Time.deltaTime);

        // Zıplama
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            velocity.y = Mathf.Sqrt(jumpHeight * -2f * gravity);
        }

        // Yerçekimi uygula
        velocity.y += gravity * Time.deltaTime;
        controller.Move(velocity * Time.deltaTime);
    }
}
```

### Hareket Yöntemleri Karşılaştırması

| Özellik | Transform | Rigidbody | CharacterController |
|---------|-----------|-----------|---------------------|
| Fizik | Yok | Tam | Kısmi |
| Çarpışma | Manuel | Otomatik | Otomatik |
| Performans | En iyi | Orta | İyi |
| Yerçekimi | Manuel | Otomatik | Manuel |
| Kullanım | Basit | Karmaşık | Orta |
| Örnek | 2D Top-down | Fizik puzzles | FPS, 3rd person |

---

## Karakter Döndürme

### Mouse ile Döndürme (FPS Tarzı)

```csharp
public class MouseLook : MonoBehaviour
{
    [SerializeField] private float sensitivity = 2f;
    [SerializeField] private float maxLookAngle = 80f;

    [SerializeField] private Transform playerBody;

    private float xRotation = 0f;

    void Start()
    {
        // Mouse'u kilitle ve gizle
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }

    void Update()
    {
        float mouseX = Input.GetAxis("Mouse X") * sensitivity;
        float mouseY = Input.GetAxis("Mouse Y") * sensitivity;

        // Yukarı/Aşağı bakış (X ekseni etrafında)
        xRotation -= mouseY;
        xRotation = Mathf.Clamp(xRotation, -maxLookAngle, maxLookAngle);

        transform.localRotation = Quaternion.Euler(xRotation, 0f, 0f);

        // Sağa/Sola dönüş (Y ekseni etrafında)
        playerBody.Rotate(Vector3.up * mouseX);
    }
}
```

### Hareket Yönüne Döndürme (3rd Person Tarzı)

```csharp
public class RotateToMovement : MonoBehaviour
{
    [SerializeField] private float rotationSpeed = 10f;

    private Vector3 moveDirection;

    void Update()
    {
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");

        moveDirection = new Vector3(h, 0, v);

        if (moveDirection.magnitude > 0.1f)
        {
            // Hareket yönüne bak
            Quaternion targetRotation = Quaternion.LookRotation(moveDirection);
            transform.rotation = Quaternion.Slerp(
                transform.rotation,
                targetRotation,
                rotationSpeed * Time.deltaTime
            );
        }
    }
}
```

---

## Kamera Kontrolü

### Kamera Takibi (Basit)

```csharp
public class CameraFollow : MonoBehaviour
{
    [SerializeField] private Transform target;
    [SerializeField] private Vector3 offset = new Vector3(0, 5, -10);
    [SerializeField] private float smoothSpeed = 5f;

    void LateUpdate()
    {
        Vector3 desiredPosition = target.position + offset;
        Vector3 smoothedPosition = Vector3.Lerp(
            transform.position,
            desiredPosition,
            smoothSpeed * Time.deltaTime
        );

        transform.position = smoothedPosition;
        transform.LookAt(target);
    }
}
```

### Orbital Kamera (3rd Person)

```csharp
public class OrbitalCamera : MonoBehaviour
{
    [SerializeField] private Transform target;
    [SerializeField] private float distance = 5f;
    [SerializeField] private float sensitivity = 3f;
    [SerializeField] private float minYAngle = -20f;
    [SerializeField] private float maxYAngle = 60f;

    private float currentX = 0f;
    private float currentY = 20f;

    void Start()
    {
        Cursor.lockState = CursorLockMode.Locked;
    }

    void LateUpdate()
    {
        currentX += Input.GetAxis("Mouse X") * sensitivity;
        currentY -= Input.GetAxis("Mouse Y") * sensitivity;
        currentY = Mathf.Clamp(currentY, minYAngle, maxYAngle);

        Quaternion rotation = Quaternion.Euler(currentY, currentX, 0);
        Vector3 position = target.position - (rotation * Vector3.forward * distance);

        transform.rotation = rotation;
        transform.position = position;
    }
}
```

### Cinemachine Kullanımı (Önerilen)

```
Window > Package Manager > Unity Registry > Cinemachine > Install
```

Cinemachine avantajları:
- Kod yazmadan kamera kontrolü
- Profesyonel kamera geçişleri
- Çoklu kamera desteği
- Hedef takibi ve çerçeveleme

```
Hierarchy > Sağ Tık > Cinemachine > Virtual Camera
```

---

## Pratik Uygulama: Tam Karakter Kontrolü

### PlayerController.cs

```csharp
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class PlayerController : MonoBehaviour
{
    [Header("Hareket Ayarları")]
    [SerializeField] private float walkSpeed = 5f;
    [SerializeField] private float runSpeed = 9f;
    [SerializeField] private float crouchSpeed = 2.5f;
    [SerializeField] private float jumpHeight = 1.5f;
    [SerializeField] private float gravity = -15f;

    [Header("Mouse Ayarları")]
    [SerializeField] private float mouseSensitivity = 2f;
    [SerializeField] private float maxLookAngle = 85f;
    [SerializeField] private Transform cameraHolder;

    [Header("Zemin Kontrolü")]
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundDistance = 0.3f;
    [SerializeField] private LayerMask groundMask;

    private CharacterController controller;
    private Vector3 velocity;
    private bool isGrounded;
    private bool isCrouching;
    private float xRotation;
    private float originalHeight;

    void Awake()
    {
        controller = GetComponent<CharacterController>();
        originalHeight = controller.height;
    }

    void Start()
    {
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }

    void Update()
    {
        GroundCheck();
        HandleMovement();
        HandleJump();
        HandleCrouch();
        HandleMouseLook();
    }

    void GroundCheck()
    {
        isGrounded = Physics.CheckSphere(groundCheck.position, groundDistance, groundMask);

        if (isGrounded && velocity.y < 0)
        {
            velocity.y = -2f;
        }
    }

    void HandleMovement()
    {
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");

        Vector3 move = transform.right * h + transform.forward * v;

        // Hız belirleme
        float currentSpeed = walkSpeed;

        if (isCrouching)
            currentSpeed = crouchSpeed;
        else if (Input.GetKey(KeyCode.LeftShift))
            currentSpeed = runSpeed;

        controller.Move(move * currentSpeed * Time.deltaTime);
    }

    void HandleJump()
    {
        if (Input.GetButtonDown("Jump") && isGrounded && !isCrouching)
        {
            velocity.y = Mathf.Sqrt(jumpHeight * -2f * gravity);
        }

        velocity.y += gravity * Time.deltaTime;
        controller.Move(velocity * Time.deltaTime);
    }

    void HandleCrouch()
    {
        if (Input.GetKeyDown(KeyCode.C))
        {
            isCrouching = !isCrouching;
            controller.height = isCrouching ? originalHeight / 2 : originalHeight;
        }
    }

    void HandleMouseLook()
    {
        float mouseX = Input.GetAxis("Mouse X") * mouseSensitivity;
        float mouseY = Input.GetAxis("Mouse Y") * mouseSensitivity;

        xRotation -= mouseY;
        xRotation = Mathf.Clamp(xRotation, -maxLookAngle, maxLookAngle);

        cameraHolder.localRotation = Quaternion.Euler(xRotation, 0f, 0f);
        transform.Rotate(Vector3.up * mouseX);
    }
}
```

### Kurulum Adımları

1. Boş bir GameObject oluştur, "Player" adını ver
2. CharacterController component'i ekle
3. PlayerController script'ini ekle
4. Player'ın altına boş GameObject oluştur: "CameraHolder"
5. Main Camera'yı CameraHolder'ın altına taşı
6. Player'ın altına boş GameObject oluştur: "GroundCheck" (ayakların altına konumla)
7. "Ground" adında Layer oluştur, zemin objelerine ata
8. Script'teki referansları Inspector'dan bağla

---

## 2D Karakter Kontrolü

### Platformer Hareketi

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody2D))]
public class Platformer2D : MonoBehaviour
{
    [Header("Hareket")]
    [SerializeField] private float moveSpeed = 8f;
    [SerializeField] private float jumpForce = 12f;

    [Header("Zemin Kontrolü")]
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundCheckRadius = 0.2f;
    [SerializeField] private LayerMask groundLayer;

    private Rigidbody2D rb;
    private bool isGrounded;
    private bool facingRight = true;

    void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        // Zemin kontrolü
        isGrounded = Physics2D.OverlapCircle(
            groundCheck.position,
            groundCheckRadius,
            groundLayer
        );

        // Zıplama
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            rb.velocity = new Vector2(rb.velocity.x, jumpForce);
        }
    }

    void FixedUpdate()
    {
        float h = Input.GetAxisRaw("Horizontal");
        rb.velocity = new Vector2(h * moveSpeed, rb.velocity.y);

        // Yön çevirme
        if ((h > 0 && !facingRight) || (h < 0 && facingRight))
        {
            Flip();
        }
    }

    void Flip()
    {
        facingRight = !facingRight;
        Vector3 scale = transform.localScale;
        scale.x *= -1;
        transform.localScale = scale;
    }

    void OnDrawGizmosSelected()
    {
        if (groundCheck != null)
        {
            Gizmos.color = Color.red;
            Gizmos.DrawWireSphere(groundCheck.position, groundCheckRadius);
        }
    }
}
```

### Top-Down Hareket (2D)

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody2D))]
public class TopDown2D : MonoBehaviour
{
    [SerializeField] private float moveSpeed = 5f;

    private Rigidbody2D rb;
    private Vector2 movement;

    void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
        rb.gravityScale = 0; // Yerçekimi kapalı
    }

    void Update()
    {
        movement.x = Input.GetAxisRaw("Horizontal");
        movement.y = Input.GetAxisRaw("Vertical");
        movement = movement.normalized; // Çapraz hızlanmayı önle

        // Mouse'a bak
        Vector3 mousePos = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        Vector2 lookDir = mousePos - transform.position;
        float angle = Mathf.Atan2(lookDir.y, lookDir.x) * Mathf.Rad2Deg - 90f;
        transform.rotation = Quaternion.Euler(0, 0, angle);
    }

    void FixedUpdate()
    {
        rb.MovePosition(rb.position + movement * moveSpeed * Time.fixedDeltaTime);
    }
}
```

---

## Mobile Input (Dokunmatik)

### Basit Touch Kontrolü

```csharp
using UnityEngine;

public class TouchInput : MonoBehaviour
{
    void Update()
    {
        // Tek parmak dokunuşu
        if (Input.touchCount > 0)
        {
            Touch touch = Input.GetTouch(0);

            switch (touch.phase)
            {
                case TouchPhase.Began:
                    Debug.Log("Dokunuldu: " + touch.position);
                    break;

                case TouchPhase.Moved:
                    Debug.Log("Sürükleniyor: " + touch.deltaPosition);
                    break;

                case TouchPhase.Ended:
                    Debug.Log("Bırakıldı");
                    break;
            }
        }
    }
}
```

### Joystick UI (On-Screen)

Yeni Input System ile:
```
GameObject > UI > On-Screen Stick
```

Veya Asset Store'dan "Joystick Pack" kullanabilirsiniz.

---

## Input Debugging

### Input Debug Script

```csharp
using UnityEngine;

public class InputDebugger : MonoBehaviour
{
    void OnGUI()
    {
        GUILayout.BeginArea(new Rect(10, 10, 300, 500));

        GUILayout.Label($"Horizontal: {Input.GetAxis("Horizontal"):F2}");
        GUILayout.Label($"Vertical: {Input.GetAxis("Vertical"):F2}");
        GUILayout.Label($"Mouse X: {Input.GetAxis("Mouse X"):F2}");
        GUILayout.Label($"Mouse Y: {Input.GetAxis("Mouse Y"):F2}");
        GUILayout.Label($"Mouse Pos: {Input.mousePosition}");

        GUILayout.Space(10);
        GUILayout.Label("Basılan Tuşlar:");

        if (Input.GetKey(KeyCode.W)) GUILayout.Label("W");
        if (Input.GetKey(KeyCode.A)) GUILayout.Label("A");
        if (Input.GetKey(KeyCode.S)) GUILayout.Label("S");
        if (Input.GetKey(KeyCode.D)) GUILayout.Label("D");
        if (Input.GetKey(KeyCode.Space)) GUILayout.Label("Space");
        if (Input.GetKey(KeyCode.LeftShift)) GUILayout.Label("Shift");

        GUILayout.EndArea();
    }
}
```

---

## Özet ve Kontrol Listesi

Bu derste öğrendiklerimiz:
- [x] Input Manager (eski sistem)
- [x] GetKey, GetKeyDown, GetKeyUp farkları
- [x] GetAxis ve GetAxisRaw kullanımı
- [x] Mouse girişleri (pozisyon, butonlar, scroll)
- [x] Yeni Input System kurulumu ve kullanımı
- [x] Transform ile hareket
- [x] Rigidbody ile hareket
- [x] CharacterController kullanımı
- [x] Mouse ile kamera döndürme
- [x] Kamera takip sistemleri
- [x] 2D karakter kontrolü
- [x] Mobile (dokunmatik) input

---

## Alıştırmalar

1. **WASD Hareketi**: Basit bir küpü WASD tuşlarıyla hareket ettirin
2. **FPS Kontrolü**: Mouse ile etrafa bakabilen karakter yapın
3. **Zıplama Mekaniği**: Çift zıplama (double jump) ekleyin
4. **Dash Sistemi**: Shift tuşu ile ileri atılma (dash) yapın
5. **2D Platformer**: Basit bir platform oyunu karakteri oluşturun
6. **Yeni Input System**: Move ve Jump action'larını tanımlayıp kullanın

---

## Faydalı İpuçları

### 1. Input Kontrolü Her Zaman Update'de
```csharp
// DOĞRU
void Update()
{
    if (Input.GetKeyDown(KeyCode.Space)) { }
}

// YANLIŞ - Input kaçabilir
void FixedUpdate()
{
    if (Input.GetKeyDown(KeyCode.Space)) { }
}
```

### 2. Hareket Yönünü Normalize Et
```csharp
Vector3 move = new Vector3(h, 0, v);
move = move.normalized; // Çapraz hareket daha hızlı olmasın
```

### 3. Time.deltaTime Kullanımı
```csharp
// Transform hareketi için
transform.Translate(direction * speed * Time.deltaTime);

// FixedUpdate'de fixedDeltaTime kullan (veya deltaTime, aynı sonuç verir)
rb.MovePosition(transform.position + direction * speed * Time.fixedDeltaTime);
```

---

## Sonraki Ders

7. Ders'te UI (User Interface) Temelleri konusunu işleyeceğiz. Canvas, Text, Button, Image ve diğer UI elemanlarını öğreneceğiz. Sağlık barı, skor göstergesi ve menü sistemleri oluşturacağız.

