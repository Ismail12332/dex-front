# Jupiter Swap Service

Микросервис на Rust для интеграции с Jupiter Aggregator API для обмена токенов в сети Solana.

## Возможности

- **POST /swap** - Создание unsigned транзакции для обмена токенов
- **GET /health** - Проверка состояния сервиса
- Интеграция с Jupiter Aggregator API
- Поддержка CORS для фронтенд приложений
- Структурированное логирование

## Быстрый старт

### Требования

- Rust 1.70+
- Cargo

### Установка и запуск

```bash
# Клонируйте репозиторий или создайте новый проект
cargo new jupiter-swap-service
cd jupiter-swap-service

# Скопируйте файлы Cargo.toml и src/main.rs

# Установите зависимости и запустите
cargo run
```

Сервис будет доступен по адресу `http://localhost:3000`

## API Endpoints

### POST /swap

Создает unsigned транзакцию для обмена токенов через Jupiter Aggregator.

**Запрос:**
```json
{
    "from_token": "So11111111111111111111111111111111111111112",
    "to_token": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "amount": 1.5,
    "user_pubkey": "7BgBvyjrZX1YKz4oh9mjb8ZScatkkwb8DzFx7LoiVkM3"
}
```

**Ответ:**
```json
{
    "tx_base64": "base64_encoded_transaction",
    "expected_output_amount": "1500000",
    "price_impact": "0.01",
    "fee": "1000",
    "last_valid_block_height": 12345678
}
```

**Параметры запроса:**

- `from_token` (string) - Адрес токена для продажи (mint address)
- `to_token` (string) - Адрес токена для покупки (mint address)  
- `amount` (float) - Количество токенов для обмена в человеческом формате
- `user_pubkey` (string) - Публичный ключ кошелька пользователя

**Параметры ответа:**

- `tx_base64` (string) - Unsigned транзакция в формате base64
- `expected_output_amount` (string) - Ожидаемое количество получаемых токенов (в lamports/smallest units)
- `price_impact` (string) - Влияние на цену в процентах
- `fee` (string) - Комиссия платформы (в lamports/smallest units)
- `last_valid_block_height` (number) - Последний валидный блок для транзакции

### GET /health

Проверка состояния сервиса.

**Ответ:** `OK`

## Примеры использования

### Обмен SOL на USDC

```bash
curl -X POST http://localhost:3000/swap \
  -H "Content-Type: application/json" \
  -d '{
    "from_token": "So11111111111111111111111111111111111111112",
    "to_token": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "amount": 0.1,
    "user_pubkey": "7BgBvyjrZX1YKz4oh9mjb8ZScatkkwb8DzFx7LoiVkM3"
  }'
```

### Обмен USDC на SOL

```bash
curl -X POST http://localhost:3000/swap \
  -H "Content-Type: application/json" \
  -d '{
    "from_token": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
    "to_token": "So11111111111111111111111111111111111111112",
    "amount": 10.0,
    "user_pubkey": "7BgBvyjrZX1YKz4oh9mjb8ZScatkkwb8DzFx7LoiVkM3"
  }'
```

## Интеграция с фронтендом

### JavaScript/TypeScript пример

```javascript
// Функция для создания swap транзакции
async function createSwapTransaction(fromToken, toToken, amount, userPubkey) {
    const response = await fetch('http://localhost:3000/swap', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            from_token: fromToken,
            to_token: toToken,
            amount: amount,
            user_pubkey: userPubkey
        })
    });

    if (!response.ok) {
        const error = await response.json();
        throw new Error(`Swap failed: ${error.message}`);
    }

    return await response.json();
}

// Пример использования с Phantom кошельком
async function swapWithPhantom() {
    try {
        // Подключение к Phantom
        const resp = await window.solana.connect();
        const userPubkey = resp.publicKey.toString();

        // Создание swap транзакции
        const swapResult = await createSwapTransaction(
            'So11111111111111111111111111111111111111112', // SOL
            'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v', // USDC
            0.1, // 0.1 SOL
            userPubkey
        );

        // Декодирование и подписание транзакции
        const transaction = Transaction.from(
            Buffer.from(swapResult.tx_base64, 'base64')
        );

        const signedTransaction = await window.solana.signTransaction(transaction);
        
        // Отправка транзакции
        const signature = await connection.sendRawTransaction(
            signedTransaction.serialize()
        );

        console.log('Transaction signature:', signature);
        console.log('Expected output:', swapResult.expected_output_amount);

    } catch (error) {
        console.error('Swap failed:', error);
    }
}
```

## Популярные адреса токенов

```
SOL:    So11111111111111111111111111111111111111112
USDC:   EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
USDT:   Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB
RAY:    4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R
SRM:    SRMuApVNdxXokk5GT7XD5cUUgXMBCoAz2LHeuAoKWRt
```

## Обработка ошибок

Сервис возвращает структурированные ошибки:

```json
{
    "error": "ERROR_CODE",
    "message": "Human readable error message"
}
```

**Возможные коды ошибок:**

- `INVALID_INPUT` - Неверные входные данные
- `INVALID_AMOUNT` - Неверное количество токенов
- `QUOTE_ERROR` - Ошибка получения котировки от Jupiter
- `SWAP_ERROR` - Ошибка создания swap транзакции

## Настройка

### Переменные окружения

```bash
# Порт сервера (по умолчанию 3000)
PORT=3000

# Уровень логирования (по умолчанию info)
RUST_LOG=info
```

### Slippage

По умолчанию используется slippage 0.5% (50 bps). Для изменения отредактируйте константу в коде:

```rust
params.insert("slippageBps", "50"); // 0.5% slippage
```

## Разработка

### Запуск в режиме разработки

```bash
cargo run
```

### Запуск тестов

```bash
cargo test
```

### Сборка для production

```bash
cargo build --release
```

## Docker

```dockerfile
FROM rust:1.70-slim as builder

WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src ./src

RUN cargo build --release

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/target/release/jupiter-swap-service /usr/local/bin/

EXPOSE 3000
CMD ["jupiter-swap-service"]
```

## Безопасность

⚠️ **Важные замечания:**

1. Этот сервис создает только unsigned транзакции
2. Приватные ключи никогда не передаются на сервер
3. Пользователь подписывает транзакции локально в своем кошельке
4. Рекомендуется использовать HTTPS в production
5. Добавьте rate limiting для production использования

## Лицензия

MIT License
