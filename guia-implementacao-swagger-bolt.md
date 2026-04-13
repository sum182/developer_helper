# Guia de Implementação: Swagger UI Bolt Standard

Este documento estabelece o padrão arquitetural para a implementação de documentação interativa (OpenAPI 3) nos projetos Bolt. O objetivo é garantir **segurança**, **independência de infraestrutura** e **facilidade de uso** para os desenvolvedores.

---

## 📚 Projetos de Referência

Para visualizar a implementação completa deste padrão, consulte os seguintes projetos:
*   **Backend**: [`varejista-api`](https://gitlab.com/boltenergy/varejista/varejista-api)
*   **Frontend**: [`varejista-web`](https://gitlab.com/boltenergy/varejista/varejista-web)

---

## 🏗️ Arquitetura da Solução

Diferente de implementações padrões, o modelo Bolt utiliza o próprio Frontend como renderizador da documentação. 

1.  **Backend**: Fornece apenas o contrato bruto (JSON) em um endpoint protegido por ambiente.
2.  **Frontend**: Atua como uma aplicação "Standalone" que consome o JSON e renderiza o Swagger UI dentro de um componente Angular, injetando automaticamente as credenciais do usuário logado.

---

## 1. Backend (Spring Boot + Kotlin)

### 1.1. Dependências
No seu `pom.xml`, adicione as bibliotecas do SpringDoc:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.7.0</version>
</dependency>
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-kotlin</artifactId>
    <version>1.7.0</version>
</dependency>
```

### 1.2. Configuração Dinâmica (`OpenAPIConfig.kt`)
Centralize os metadados e utilize a propriedade `services.host` já existente no projeto para evitar URLs fixas.

```kotlin
@Configuration
class OpenAPIConfig {
    @Value("\${services.host}")
    private lateinit var host: String

    @Bean
    fun customOpenAPI(): OpenAPI {
        return OpenAPI()
            .info(Info()
                .title("API Name")
                .description("Technical description of the service")
                .contact(Contact().name("Bolt Dev Team").email("sistemas@boltenergy.com.br")))
            .servers(listOf(
                Server().url(host).description("Current Environment Server")
            ))
    }
}
```

### 1.3. Segurança (`WebSecurityConfig.kt`)
Libere as rotas apenas quando o Swagger estiver habilitado via configuração, garantindo que o Spring Security não redirecione a chamada para a página de login HTML ao ser solicitada pelo front.

```kotlin
@Value("\${springdoc.api-docs.enabled:false}")
private var isSwaggerEnabled: Boolean = false

// Dentro do filterChain
if (isSwaggerEnabled) {
    permittedUrls.addAll(listOf("/v3/api-docs/**", "/swagger-ui/**"))
}
```

### 1.4. Configuração por Ambiente (`application.yml`)
O Swagger deve estar desativado por padrão e ativo apenas nos perfis de desenvolvimento.

**application.yml (Padrão/Produção):**
```yaml
springdoc:
  api-docs:
    enabled: false
    path: /api/v1/api-docs # Prefixo /api/v1 ajuda a passar pelas regras de Ingress existentes
  swagger-ui:
    enabled: false
```

---

## 2. Frontend (Angular)

### 2.1. Instalação
```bash
npm install swagger-ui-dist
npm install path-browserify --save-dev # Necessário para compatibilidade Webpack 5+
```

### 2.2. Ajuste de Build (`tsconfig.json`)
Mapeie o polyfill do módulo `path` para evitar erros de compilação durante o build.

**tsconfig.json:**
```json
"compilerOptions": {
  "paths": {
    "path": ["./node_modules/path-browserify"]
  }
}
```

### 2.3. Componente de Documentação (`SwaggerComponent.ts`)
Crie um componente que realize as seguintes tarefas automáticas:

1.  **Proteção de Rota**: Redireciona para o login se o usuário não tiver token.
2.  **Injeção de Auth**: Injeta o `Authorization: Bearer` automaticamente em todas as chamadas.
3.  **Carregamento de CSS (CI/CD Safe)**: Injeta o CSS via link programático para evitar erros de processamento de SVG no Pipeline do GitLab.
4.  **Performance**: Desativa validadores externos (`validatorUrl: null`) para funcionar em redes privadas.

```typescript
// Exemplo de carregamento de CSS seguro para GitLab Pipeline
private loadSwaggerCss(): void {
  this.styleTag = document.createElement('link');
  this.styleTag.rel = 'stylesheet';
  this.styleTag.href = 'https://unpkg.com/swagger-ui-dist@5.11.0/swagger-ui.css';
  document.head.appendChild(this.styleTag);
}

// Inicialização do Bundle
SwaggerUIBundle({
  url: `${environment.api}/api/v1/api-docs`,
  domNode: this.swaggerUiElement.nativeElement,
  validatorUrl: null,
  presets: [SwaggerUIBundle.presets.apis, SwaggerUIStandalonePreset],
  layout: 'BaseLayout',
  requestInterceptor: (request) => {
    request.headers['Authorization'] = `Bearer ${token}`;
    return request;
  }
});
```

---

## ✅ Vantagens do Padrão Bolt

*   **Independência de Infra**: Não requer que o time de SDR crie regras de Proxy Reverso no Nginx ou libere portas extras no Ingress.
*   **Pipeline Seguro**: A técnica de injeção dinâmica de CSS evita erros comuns de Parsing (SVG) em runners Linux.
*   **Segurança Corporativa**: A documentação não fica pública; ela herda a segurança do portal e exige token JWT válido.
*   **Consistência**: Todos os projetos da Bolt utilizam a mesma interface e o mesmo fluxo de acesso.
