# Qual a diferença entre usar um Converter (binding) na Web e usar Jackson (serialização)?

Este documento é para estudo e consulta rápida sobre as diferenças entre implementar a conversão de uma String para `TSID` no binding do Spring MVC (ex.: `Converter<String, TSID>`) versus implementar (de)serialização usando Jackson (JSON). Ele explica escopo, quando cada abordagem é usada, como registrar, erros comuns, recomendações e exemplos práticos.

## Objetivo

Explicar de forma prática e direta quando usar cada abordagem e por quê — especialmente no contexto do `StringToTSIDWebConverter` presente no projeto.

## Resumo rápido

- Converter (Spring `Converter` / `Formatter`) = usado no binding de parâmetros web: path variables, query params, form data e parâmetros de métodos do controller.
- Jackson (`JsonSerializer` / `JsonDeserializer`) = usado na (de)serialização de bodies JSON (`@RequestBody` / `@ResponseBody`).

Use os dois quando a sua aplicação precisa tratar `TSID` tanto em parâmetros de rota/consulta quanto em bodies JSON.

---

## Diferenças detalhadas

### Escopo
- Converter: atua no nível do Spring MVC, antes do controller, convertendo Strings vindas da web para tipos fortes (como `TSID`). Afeta `@PathVariable`, `@RequestParam`, `@ModelAttribute` e binding de formulários.
- Jackson: atua no nível do mapeador JSON. Afeta apenas quando o payload é JSON — ou seja, `@RequestBody` e `@ResponseBody` (ou qualquer uso do `ObjectMapper`).

### Quando é usado
- Converter: quando o cliente envia um `TSID` como parte da URL (`/devices/{id}`) ou como query param `?deviceId=...`.
- Jackson: quando o cliente envia/recebe JSON contendo campos que representam `TSID`.

### Registro
- Converter: registra-se no `WebConfig` (implementando `WebMvcConfigurer` e sobrescrevendo `addFormatters` ou `addConverters`).
- Jackson: registra-se no `ObjectMapper` como um módulo, via `Jackson2ObjectMapperBuilder`, `@JsonSerialize`/`@JsonDeserialize` nas classes ou registrando globalmente um `Module`.

### Erros e validação
- Converter: pode retornar `null` ou lançar `IllegalArgumentException`. Erros no converter geralmente viram 400 (Bad Request) quando o Spring não consegue fazer o binding do parâmetro.
- Jackson: erros de desserialização (por exemplo, `JsonParseException` / `JsonMappingException`) também resultam em 400; porém a mensagem de erro e o local (body JSON versus parâmetro de rota) diferem.

### Impacto no fluxo
- Converter atua mais cedo (antes do controller) e é ideal para entradas vindas da URL/query/form.
- Jackson só entra em cena quando há JSON para (de)serializar.

---

## Recomendações práticas

- Use `Converter<String, TSID>` para binding de rota/query/form (por ex. `StringToTSIDWebConverter`).
- Use `JsonSerializer`/`JsonDeserializer` para suportar `TSID` em bodies JSON.
- Se a sua aplicação usa `TSID` em ambos os lugares, implemente ambos (converter + serializer/deserializer). Isso garante comportamento consistente e mensagens de erro claras conforme o contexto.

---

## Exemplos (práticos)

### Melhorias sugeridas para `StringToTSIDWebConverter` (binding web)



- Observação: lançar `IllegalArgumentException` resulta em 400 e uma mensagem de validação mais clara para o cliente.

### Jackson: serializador e desserializador para `TSID`

```java
// Serializador (exemplo já presente no projeto)
public class TSIDToStringSerializer extends JsonSerializer<TSID> {
    @Override
    public void serialize(TSID value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeString(value.toString());
    }
}

// Desserializador
public class StringToTSIDDeserializer extends JsonDeserializer<TSID> {
    @Override
    public TSID deserialize(JsonParser p, DeserializationContext ctxt) throws IOException {
        String text = p.getValueAsString();
        if (text == null || text.isBlank()) {
            return null;
        }
        try {
            return TSID.from(text);
        } catch (Exception ex) {
            throw new JsonMappingException(p, "TSID inválido: " + text, ex);
        }
    }
}
```

Como registrar no `ObjectMapper`:

```java
SimpleModule tsidModule = new SimpleModule();
ts idModule.addSerializer(TSID.class, new TSIDToStringSerializer());
tsidModule.addDeserializer(TSID.class, new StringToTSIDDeserializer());
objectMapper.registerModule(tsidModule);
```

Ou configure via Spring Boot `Jackson2ObjectMapperBuilder`/`Jackson2Module` para registro automático.


## FAQ rápido

- Preciso implementar ambos? R: Sim, se sua API aceita `TSID` tanto em URLs/params quanto em JSON, e fora da api você quer tratar esse TDIS como String
- Posso centralizar numa biblioteca? R: Sim — criar um módulo utilitário que exponha o `Converter` e o módulo Jackson é uma boa prática.

---

Arquivo criado para consultas futuras.

