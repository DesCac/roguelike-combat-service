# Combat Service (Go)

**Validates attacks and calculates damage** в roguelike игре.

## Ответственность

✅ **ЧТО ДЕЛАЕТ:**
- Attack validation (range, cooldown)
- Damage calculation (base + weapon bonus)
- Defense calculation (armor bonus)
- Health updates
- Death detection
- Публикация `entity.died` событий в NATS

❌ **ЧТО НЕ ДЕЛАЕТ:**
- Movement validation
- AI логика
- Quest triggers (делается асинхронно через Event Bus)

## Технологии

- **Язык:** Go 1.21+
- **gRPC:** google.golang.org/grpc
- **Event Bus:** NATS JetStream
- **Протоколы:** contracts/proto/combat/

## Размер кода

**Target:** ~800 LOC

## gRPC Contract

```protobuf
service CombatService {
  rpc ValidateAttack(ValidateAttackRequest) returns (ValidateAttackResponse);
}
```

## Запуск локально

```bash
# Install dependencies
go mod download

# Run service
go run main.go

# Health check
grpcurl -plaintext localhost:50052 grpc.health.v1.Health/Check
```

## Docker

```bash
# Build
docker build -t combat-service:dev -f Dockerfile.dev .

# Run
docker run -p 50052:50052 --env NATS_URL=nats://nats:4222 combat-service:dev
```

## Environment Variables

```bash
GRPC_PORT=50052
NATS_URL=nats://localhost:4222
LOG_LEVEL=info
```

## Testing

```bash
go test ./...
```

## Примеры использования

### Успешная атака
```json
Request:
{
  "game_id": "game_123",
  "attacker_id": "player_1",
  "target_id": "mob_123",
  "attacker_stats": {
    "base_damage": 10,
    "equipment": {
      "weapon": { "stats": { "damage_bonus": 5 } }
    }
  },
  "target_stats": {
    "current_health": 100,
    "equipment": {
      "armor": { "stats": { "defense_bonus": 2 } }
    }
  }
}

Response:
{
  "status": "SUCCESS",
  "damage_dealt": 13,  // 10 + 5 - 2
  "target_health_after": 87,
  "target_died": false
}
```

## Event Publishing (Async)

После успешной атаки публикует событие:
```json
Topic: "game.combat.damage_dealt"
Payload: {
  "game_id": "game_123",
  "attacker_id": "player_1",
  "target_id": "mob_123",
  "damage": 13,
  "target_health_after": 87,
  "target_died": false
}
```

## Performance

- **Target latency:** < 10ms (p95)
- **Throughput:** ~8,000 attacks/sec

---

**Статус:** Skeleton (требует имплементации)
**Следующие шаги:** Реализовать ValidateAttack с damage calculation и event publishing
