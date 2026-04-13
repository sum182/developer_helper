# Padrão de Implementação: Spring Data JPA (Filtros, Paginação e Ordenação)

Este documento serve como referência técnica para novas implementações de listagem no projeto, utilizando os recursos nativos do Spring Data JPA para garantir performance e escalabilidade.

---

## 1. Camada de Filtro (DTO)
Crie um DTO dedicado para capturar os parâmetros de busca. Use anotações do Swagger para documentação e `@DateTimeFormat` para campos de data.

```kotlin
@Schema(description = "Filtros para pesquisa")
class ExemploFiltroDto {
    var nome: String? = null
    var status: Enum? = null

    @field:DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
    var dataInicio: LocalDate? = null
}
```

## 2. Camada de Persistência (Repository)
O Repository deve estender `JpaSpecificationExecutor` para habilitar consultas dinâmicas.

```kotlin
@Repository
interface ExemploRepository : JpaRepository<Entidade, Int>, JpaSpecificationExecutor<Entidade>
```

## 3. Camada de Especificação (Specification)
Utilize a Criteria API para construir predicados dinâmicos. Isso evita SQL manual e previne SQL Injection.

```kotlin
object ExemploSpecification {
    fun search(filtro: ExemploFiltroDto): Specification<Entidade> {
        return Specification { root, _, cb ->
            val predicates = mutableListOf<Predicate>()

            if (!filtro.nome.isNullOrBlank()) {
                predicates.add(cb.like(cb.lower(root.get("nome")), "%${filtro.nome!!.lowercase()}%"))
            }

            if (filtro.dataInicio != null) {
                predicates.add(cb.greaterThanOrEqualTo(root.get("data"), filtro.dataInicio))
            }

            cb.and(*predicates.toTypedArray())
        }
    }
}
```

## 4. Camada de Serviço (Service)
O serviço orquestra a busca, converte o resultado para DTO e, se necessário, aplica lógicas de negócio complementares sobre a lista resultante.

```kotlin
fun list(filtro: ExemploFiltroDto, pageable: Pageable): PageResponse<ExemploDto> {
    val specification = ExemploSpecification.search(filtro)
    
    // Executa busca paginada e converte para DTO
    val page = exemploRepository.findAll(specification, pageable).map { ExemploDto(it) }

    // Aplica lógica complementar (opcional)
    // processarDadosAdicionais(page.content)

    return PageResponse(page)
}
```

## 5. Camada de Controle (Controller)
Utilize `@ModelAttribute` para os filtros e `@PageableDefault` para definir o comportamento padrão de ordenação e tamanho de página.

```kotlin
@GetMapping("/api/v1/recurso")
fun list(
    @ModelAttribute filtro: ExemploFiltroDto,
    @PageableDefault(size = 20, sort = ["id"], direction = Sort.Direction.DESC) pageable: Pageable
): PageResponse<ExemploDto> {
    return exemploService.list(filtro, pageable)
}
```

## 6. Vantagens deste Padrão
1.  **Performance**: Paginação feita via `LIMIT` e `OFFSET` direto no banco de dados.
2.  **Flexibilidade**: O frontend pode alterar a ordenação via URL (`?sort=nome,asc`) sem mudar o backend.
3.  **Manutenibilidade**: Código limpo, tipado e seguindo os padrões oficiais do Spring Boot.
4.  **Consistência**: Todas as listagens do sistema passam a ter o mesmo comportamento e estrutura de resposta.
