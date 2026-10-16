# Desafio Prompt Engineering

Desafio de **Prompt Engineering** com 4 questões, usando o ChatGPT como ferramenta de IA generativa. Para cada questão foi escolhida a técnica de prompt mais adequada ao tipo de resposta esperada.

## Sumário

| # | Questão | Técnica aplicada |
|---|---------|------------------|
| 1 | Ray Tracing: cálculo da cor de um pixel | Persona + contexto + resposta estruturada |
| 2 | Decomposição numérica de 142.981 | Chain-of-Thought + desambiguação |
| 3 | Personagens de As Crônicas de Gelo e Fogo × Maquiavel | Few-shot + restrição de fidelidade |
| 4 | Endpoint FastAPI com validação de `Item` | Especificação detalhada de requisitos |

---

## Questão 1: Ray Tracing

**Objetivo:** entender como o algoritmo de Ray Tracing calcula a cor de um pixel em uma imagem renderizada.

**Técnica:** Persona + contexto do público + resposta estruturada em etapas.

**Por que essa técnica:** é um tema técnico e com várias etapas. Definir quem explica e para quem, e forçar uma estrutura, evita respostas genéricas ou desorganizadas.

**Prompt:**

```
Você é um professor de computação gráfica explicando para um estudante
de Análise e Desenvolvimento de Sistemas, que conhece programação mas
não tem base avançada de física.

Explique como o algoritmo de Ray Tracing calcula a cor de um pixel em
uma imagem renderizada. Estruture a resposta em etapas numeradas:
1. Geração do raio primário (câmera → pixel)
2. Teste de interseção com os objetos da cena
3. Cálculo da iluminação no ponto de interseção (difusa, especular, sombras)
4. Raios secundários (reflexão e refração) e recursão
5. Combinação final e cor do pixel

Inclua as fórmulas principais (ex.: modelo de Phong) e um pseudocódigo
curto. Termine com um resumo de 3 linhas.
```

---

## Questão 2: Decomposição numérica de 142.981

**Objetivo:** obter uma resposta completa de como fazer a decomposição numérica de 142.981.

**Técnica:** Chain-of-Thought (raciocínio passo a passo) + desambiguação.

**Por que essa técnica:** "decomposição numérica" pode significar decomposição posicional ou fatoração em primos. O prompt cobre as duas interpretações e exige a demonstração de cada passo, além de uma verificação final.

**Prompt:**

```
Quero a decomposição numérica completa do número 142.981.
Resolva passo a passo, mostrando o raciocínio de cada etapa:

A) Decomposição posicional: separe por ordem (unidades, dezenas,
   centenas, etc.) e escreva como soma de valores posicionais.
B) Fatoração em números primos: teste a divisibilidade por primos
   sucessivos até a raiz quadrada do número, mostrando cada teste.
   Se o número for primo, diga isso explicitamente e justifique.

Ao final, confira o resultado com uma verificação (por exemplo,
multiplicando os fatores ou somando os valores posicionais).
```

> **Observação:** como LLMs podem errar contas, o resultado deve ser conferido (por exemplo, com Python).

---

## Questão 3: Personagens de As Crônicas de Gelo e Fogo × Maquiavel

**Objetivo:** identificar personagens com características inspiradas na filosofia de Maquiavel.

**Técnica:** Few-shot (exemplo do formato de saída) + restrição de fidelidade.

**Por que essa técnica:** o exemplo padroniza a resposta, e a instrução de não inventar detalhes reduz alucinações em um tema que depende de fatos da obra.

**Prompt:**

```
Atue como analista literário especialista em As Crônicas de Gelo e
Fogo (George R. R. Martin) e em filosofia política, especialmente
O Príncipe, de Maquiavel.

Liste os personagens cujas características, decisões ou estratégias
dialogam com ideias maquiavélicas (virtù, fortuna, "é melhor ser
temido que amado", fim justifica os meios, aparência vs. realidade).

Para cada personagem, siga este formato:
- Personagem:
- Conceito maquiavélico relacionado:
- Exemplo concreto na história:

Exemplo do formato:
- Personagem: Tywin Lannister
- Conceito: uso da força e do medo para manter o poder
- Exemplo: As Bodas Vermelhas / a Chuva de Castamere

Use apenas eventos que existem nos livros ou na série. Se não tiver
certeza sobre um detalhe, diga que não tem certeza em vez de inventar.
```

---

## Questão 4: Endpoint FastAPI

**Objetivo:** criar um endpoint com FastAPI que valide e processe um objeto `Item`.

**Técnica:** Especificação detalhada (requisitos explícitos + formato de entrega).

**Estrutura do `Item`:**

| Campo | Tipo | Regra |
|-------|------|-------|
| `nome` | `string` | Máximo de 25 caracteres |
| `valor` | `float` | n/a |
| `data` | `date` | Não pode ser superior à data atual |

**Requisitos:**

- O endpoint valida os valores recebidos.
- Após a validação, o `Item` é retornado com um novo campo `uuid`, gerado dinamicamente (UUID4).

**Prompt:**

```
Crie um endpoint com FastAPI (Python 3.10+, Pydantic v2) com estas
especificações:

Modelo Item:
- nome: string, máximo 25 caracteres
- valor: float
- data: date, não pode ser superior à data atual

Requisitos:
- Endpoint POST /items que valida o corpo da requisição.
- Em caso de dados inválidos, retornar erro 422 com mensagem clara.
- Após validar, retornar o Item recebido com um campo adicional "uuid"
  (UUID4 gerado dinamicamente).

Entregue: código completo em um único arquivo main.py, comando para
rodar com uvicorn e um exemplo de requisição com curl (válida e inválida).
```

### Código (`main.py`)

```python
from datetime import date
from uuid import UUID, uuid4

from fastapi import FastAPI
from pydantic import BaseModel, Field, field_validator

app = FastAPI()


class Item(BaseModel):
    nome: str = Field(..., max_length=25)
    valor: float
    data: date

    @field_validator("data")
    @classmethod
    def data_nao_futura(cls, v: date) -> date:
        if v > date.today():
            raise ValueError("A data não pode ser superior à data atual.")
        return v


class ItemResponse(Item):
    uuid: UUID


@app.post("/items", response_model=ItemResponse)
def criar_item(item: Item):
    return ItemResponse(**item.model_dump(), uuid=uuid4())
```

### Como rodar

```bash
pip install fastapi uvicorn
uvicorn main:app --reload
```

Documentação interativa (Swagger): http://127.0.0.1:8000/docs

### Exemplos de requisição

**Válida:**

```bash
curl -X POST http://127.0.0.1:8000/items \
  -H "Content-Type: application/json" \
  -d '{"nome": "Teclado", "valor": 199.90, "data": "2026-10-01"}'
```

Resposta esperada (o `uuid` muda a cada chamada):

```json
{
  "nome": "Teclado",
  "valor": 199.9,
  "data": "2026-10-01",
  "uuid": "3f2b8c1e-5a47-4d0b-9c1a-7e6f2d4b8a10"
}
```

**Inválida (data futura):**

```bash
curl -X POST http://127.0.0.1:8000/items \
  -H "Content-Type: application/json" \
  -d '{"nome": "Teclado", "valor": 199.90, "data": "2099-01-01"}'
```

Retorna `422 Unprocessable Entity` com a mensagem de validação.

---

## Autor

**Davi Rodrigues dos Santos Cariolano** ([@TrevoCode](https://github.com/TrevoCode))
