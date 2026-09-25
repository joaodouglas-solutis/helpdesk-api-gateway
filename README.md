# Helpdesk API Gateway

Gateway responsável pelo ponto único de entrada HTTP da aplicação.

## Arquitetura

O Gateway utiliza Spring Cloud Gateway com WebFlux.

O fluxo externo é:

```text
Frontend
   ↓
API Gateway :8080
   ↓
┌──────────────┬──────────────┬──────────────────┐
│ User Service │ Ticket       │ Notification     │
│ :8081        │ Service :8082│ Service :8083    │
└──────────────┴──────────────┴──────────────────┘
```

As rotas são configuradas no `application.yaml`.

## Roteamento

Os prefixos públicos são:

```text
/api/auth/**
/api/users/**
/api/tickets/**
/api/notifications/**
```

Cada prefixo possui seu destino configurável por variável de ambiente:

```text
USER_SERVICE_URL
TICKET_SERVICE_URL
NOTIFICATION_SERVICE_URL
```

Existem valores padrão para execução local.

## StripPrefix

As rotas utilizam:

```text
StripPrefix=1
```

Isso permite que o frontend trabalhe com:

```text
/api/users
```

enquanto o serviço interno receba:

```text
/users
```

O mesmo padrão é aplicado aos demais serviços.

## Segurança reativa

A configuração utiliza:

```java
@EnableWebFluxSecurity
```

e:

```java
SecurityWebFilterChain
```

em vez da configuração tradicional baseada em Servlet.

O JWT é processado através de:

```java
NimbusReactiveJwtDecoder
```

e o conversor é adaptado para o modelo reativo através de:

```java
ReactiveJwtAuthenticationConverterAdapter
```

## Autorização centralizada

O Gateway possui regras específicas para:

```text
/api/auth/**
/api/users/register
/api/users/clients
/api/users/*/summary
/api/users/**
/api/tickets/**
/api/notifications/**
```

A autenticação dos serviços de tickets e notificações é exigida no Gateway antes do encaminhamento.

## CORS

O Gateway também funciona como ponto de controle CORS.

A origem configurada atualmente é:

```text
http://localhost:5173
```

Os métodos aceitos incluem:

```text
GET
POST
PUT
PATCH
DELETE
OPTIONS
```

A configuração está presente tanto na camada de segurança quanto na configuração global do Spring Cloud Gateway.

## Health Check

O endpoint:

```text
/actuator/health
```

é liberado para permitir verificação da disponibilidade do Gateway sem autenticação.

## Configuração externa

As URLs dos microsserviços não ficam rigidamente acopladas ao código.

Exemplo:

```yaml
uri: ${USER_SERVICE_URL:http://localhost:8081}
```

A mesma abordagem é usada para Ticket Service e Notification Service.

O segredo JWT também é parametrizado:

```yaml
secret: ${JWT_SECRET:...}
```

## Containerização

O repositório possui Dockerfile e Maven Wrapper, mantendo o Gateway preparado para execução containerizada e sem depender de uma instalação global do Maven.
