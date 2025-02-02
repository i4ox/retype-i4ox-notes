# Пакет crypto

Пакет `crypto` предоставляет набор интерфейсов и функций для работы с криптографическими операциями, такими как хеширование, шифрование, цифровые подписи и генерация случайных чисел.

Этот пакет является частью стандартной библиотеки Go и служит основой для более специализированных пакетов, таких как `crypto/aes`, `crypto/md5`, `crypto/rsa`, `crypto/sha1`, `crypto/sha256` и другие.

## Основные возможности

### Хеширование

Пакет `crypto` предоставляет функции для работы с хешами SHA-256.

Примеры хеш-функций:

- `crypto/sha256` - SHA-256 (256-битный хэш-функция).
- `crypto/sha512` - SHA-512 (512-битный хэш-функция).
- `crypto/md5` - MD5 (128-битный хэш-функция) - **Не рекомендуется использовать в криптографических целях**.

Часто используемые функции:

- `sha256.New()` - Создает новый хэшер SHA-256.
- `sha256.Sum256(data []byte) [32]byte` - Рассчитывает SHA-256 хэш данных.

```go
import (
	"crypto/sha256"
	"fmt"
)

func main() {
	data := []byte("hello world")
	hash := sha256.Sum256(data)
	fmt.Printf("SHA-256: %x\n", hash)
}
```

### Симметричное шифрование

Пакет `crypto` предоставляет интерфейсы для работы с симметричными шифрованиями, такие как `cipher.Block`, `cipher.Stream`.

Примеры алгоритмов шифрования:

- `crypto/aes` - AES (Advanced Encryption Standard).
- `crypto/des` - DES (Data Encryption Standard) - **Устарело**.

Часто используемые функции:

- `aes.NewCipher(key []byte) (cipher.Block, error)` - Создает новый блок AES.
- `cipher.NewCBCEncrypter(block cipher.Block, iv []byte) cipher.BlockMode` — создает шифратор в режиме CBC.
- `cipher.NewCBCDecrypter(block cipher.Block, iv []byte) cipher.BlockMode` — создает дешифратор в режиме CBC.

!!!
Режим CBC — это шифрование в цикле, которое позволяет использовать блок шифрования для каждого блока данных.
!!!

```go
import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"fmt"
	"io"
)

// Паддинг PKCS#7 - нужен так как блоки должны быть кратны 16 байт
func padPKCS7(data []byte, blockSize int) []byte {
	padding := blockSize - (len(data) % blockSize)
	padText := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(data, padText...)
}

// Убираем паддинг PKCS#7
func unpadPKCS7(data []byte) []byte {
	length := len(data)
	unpadding := int(data[length-1])
	return data[:(length - unpadding)]
}

func main() {
	key := []byte("examplekey123456")
	plaintext := []byte("hello world")

	block, err := aes.NewCipher(key)
	if err != nil {
		panic(err)
	}

	iv := make([]byte, aes.BlockSize)
	if _, err := io.ReadFull(rand.Reader, iv); err != nil {
		panic(err)
	}

	paddedText := padPKCS7(plaintext, aes.BlockSize)

	ciphertext := make([]byte, len(paddedText))
	mode := cipher.NewCBCEncrypter(block, iv)
	mode.CryptBlocks(ciphertext, paddedText)

	fmt.Printf("Ciphertext: %x\n", ciphertext)

	mode = cipher.NewCBCDecrypter(block, iv)
	mode.CryptBlocks(ciphertext, ciphertext)

	plaintext = unpadPKCS7(ciphertext)

	fmt.Printf("Plaintext: %s\n", plaintext)
}
```

### Ассиметричное шифрование

Пакет `crypto` предоставляет интерфейсы для асимметричного шифрования, такие как `crypto.PublicKey` и `crypto.PrivateKey`.

Примеры алгоритмов шифрования:

- `crypto/rsa` - RSA (Rivest–Shamir–Adleman).
- `crypto/dsa` - DSA (Digital Signature Algorithm).
- `crypto/ecdsa` - ECDSA (Elliptic Curve Digital Signature Algorithm).

Часто используемые функции:

- `rsa.GenerateKey(rand io.Reader, bitSize int) (*rsa.PrivateKey, error)` - Создает новый ключ RSA.
- `rsa.EncryptPKCS1v15(rand io.Reader, pub *rsa.PublicKey, plaintext []byte) ([]byte, error)` - Шифрует данные с использованием RSA с использованием PKCS#1.
- `rsa.DecryptPKCS1v15(rand io.Reader, priv *rsa.PrivateKey, ciphertext []byte) ([]byte, error)` - Расшифровывает данные с использованием RSA с использованием PKCS#1.

```go
import (
	"crypto/rand"
	"crypto/rsa"
	"fmt"
)

func main() {
	privateKey, err := rsa.GenerateKey(rand.Reader, 2048)
	if err != nil {
		panic(err)
	}
	publicKey := &privateKey.PublicKey

	ciphertext, err := rsa.EncryptPKCS1v15(rand.Reader, publicKey, []byte("hello world"))
	if err != nil {
		panic(err)
	}
	fmt.Printf("Ciphertext: %x\n", ciphertext)

	plaintext, err := rsa.DecryptPKCS1v15(rand.Reader, privateKey, ciphertext)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Plaintext: %s\n", plaintext)
}
```

### Цифровые подписи

Пакет `crypto` предоставляет интерфейсы для создания и проверки цифровых подписей.

Примеры алгоритмов цифровой подписи:

- `crypto/rsa` - RSA (Rivest–Shamir–Adleman).
- `crypto/ecdsa` - ECDSA (Elliptic Curve Digital Signature Algorithm).

Часто используемые функции:

- `ecdsa.GenerateKey(curve elliptic.Curve, rand io.Reader) (*ecdsa.PrivateKey, error)` - Создает новый ключ ECDSA.
- `ecdsa.Sign(rand io.Reader, priv *ecdsa.PrivateKey, hash []byte) (r, s *big.Int, err error)` - Подписывает данные с использованием ECDSA.
- `ecdsa.Verify(pub *ecdsa.PublicKey, hash []byte, r, s *big.Int) bool` - Проверяет подпись ECDSA.

```go
import (
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"fmt"
)

func main() {
	privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	if err != nil {
		panic(err)
	}

	msg := "hello world"
	hash := sha256.Sum256([]byte(msg))
	r, s, err := ecdsa.Sign(rand.Reader, privateKey, hash[:])
	if err != nil {
		panic(err)
	}

	valid := ecdsa.Verify(&privateKey.PublicKey, hash[:], r, s)
	fmt.Println("Signature valid:", valid)
}
```

### Генерация случайных чисел

Пакет `crypto/rand` предоставляет криптографически безопасный генератор случайных чисел.

Часто используемые функции:

- `rand.Read(b []byte)` - Создает случайные числа в буфере `b`.
- `rand.Intn(n int) int` - Возвращает случайное число от 0 до `n-1`.

```go
import (
	"crypto/rand"
	"fmt"
)

func main() {
	b := make([]byte, 16)
	_, err := rand.Read(b)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%x\n", b)
}
```

### Остальные возможности

- Поддержка TLS/SSL.
- Поддержка X.509 сертификатов.
- Поддержка других криптографических стандартов.
