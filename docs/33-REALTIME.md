# 33 - Comunicación en Tiempo Real

> Para notificaciones, chat, dashboards en vivo y colaboración multi-usuario.

---

## Cuándo Usar Real-Time

| FEATURE                  | NECESITA REAL-TIME | TÉCNICA RECOMENDADA |
|-------------------------|--------------------|-----------------------|
| Chat 1-a-1               | Sí                 | WebSockets (Socket.io) |
| Notificaciones push      | Sí                 | SSE o WebSockets |
| Dashboard métricas       | Sí                 | SSE |
| Colaboración (Docs)      | Sí                 | WebSockets + CRDT |
| Precio de acciones       | Sí                 | SSE |
| Lista de productos       | No                 | HTTP Fetch normal |

---

## Estrategia Angular (RxJS)

En Angular, modelamos el real-time como **Streams de Datos (Observables)**.

### 1. WebSockets (Socket.io + NgxSocketIo)

**Instalación:** `npm i ngx-socket-io`

```typescript
// socket.service.ts
import { Injectable } from '@angular/core';
import { Socket } from 'ngx-socket-io';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class ChatService {
  constructor(private socket: Socket) {}

  sendMessage(msg: string) {
    this.socket.emit('message', msg);
  }

  getMessage(): Observable<string> {
    return this.socket.fromEvent<string>('message');
  }
}
```

**Uso en Componente (AsyncPipe):**
```typescript
@Component({
  template: `
    <ul>
      @for (msg of messages$ | async; track $index) {
        <li>{{ msg }}</li>
      }
    </ul>
    <input #input (keyup.enter)="send(input.value)">
  `
})
export class ChatComponent {
  messages$ = this.chatService.getMessage();

  constructor(private chatService: ChatService) {}

  send(msg: string) {
    this.chatService.sendMessage(msg);
  }
}
```

### 2. Server-Sent Events (SSE)

Ideal para notificaciones unidireccionales (Servidor -> Cliente).
Usamos `EventSource` nativo envuelto en un `Observable`.

```typescript
// sse.service.ts
import { Injectable, NgZone } from '@angular/core';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class SseService {
  constructor(private zone: NgZone) {}

  getServerSentEvent(url: string): Observable<MessageEvent> {
    return new Observable(observer => {
      const eventSource = new EventSource(url);
      
      eventSource.onmessage = event => {
        this.zone.run(() => observer.next(event));
      };
      
      eventSource.onerror = error => {
        this.zone.run(() => observer.error(error));
      };
      
      return () => eventSource.close();
    });
  }
}
```

**Backend (NestJS SSE):**
```typescript
@Sse('sse')
sse(): Observable<MessageEvent> {
  return interval(1000).pipe(map((_) => ({ data: { hello: 'world' } })));
}
```

---

## WebSockets con NestJS

```typescript
// chat.gateway.ts
@WebSocketGateway({ cors: true })
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('message')
  handleMessage(@MessageBody() message: string): void {
    this.server.emit('message', message);
  }
}
```

---

## Escalabilidad (Redis Adapter)

Si tienes múltiples instancias de backend (Docker Swarm / K8s), necesitas Redis para sincronizar los sockets.

```typescript
// main.ts
const redisIoAdapter = new RedisIoAdapter(app);
await redisIoAdapter.connectToRedis();
app.useWebSocketAdapter(redisIoAdapter);
```

**Por qué Redis Adapter:**
- Permite múltiples instancias del servidor
- Los mensajes se comparten entre todas las instancias
- Esencial para escalar horizontalmente
