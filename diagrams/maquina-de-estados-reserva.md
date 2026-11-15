# Máquina de estados del proceso de reserva

```mermaid
stateDiagram-v2
    [*] --> STARTED
    STARTED --> HELD
    STARTED --> FAILED

    HELD --> EN_CURSO
    WAITING_FOR_PHONE --> CONFIRMED
    PHONE_REQUESTED --> CONFIRMED
    PHONE_SUBMITTED --> CONFIRMED
    CONFIRMED --> CANCELLED

    EN_CURSO --> EXPIRED
    EN_CURSO --> INVALID
    EN_CURSO --> FAILED

    CANCELLED --> [*]
    EXPIRED --> [*]
    INVALID --> [*]
    FAILED --> [*]
```

Desde cualquier estado en curso (HELD, WAITING_FOR_PHONE, PHONE_REQUESTED, PHONE_SUBMITTED y PHONE_REJECTED) el proceso
puede pasar a EXPIRED, INVALID o FAILED. Los estados finales no tienen salida. CONFIRMED solo admite CANCELLED.