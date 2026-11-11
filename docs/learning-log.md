# Diário de aprendizado

Registro do que fui descobrindo ao longo do projeto: problemas, soluções e decisões. Entradas mais recentes no topo.

---

## 11-11-2026 · Valhalla no ar e o primeiro problema real

### O que fiz

- Configurei o ambiente: JDK 17, Docker, Docker Compose e Git.
- Criei o repositório e a estrutura de pastas do projeto.
- Subi o **Valhalla** (motor de rotas) via Docker Compose, usando a imagem `gis-ops/docker-valhalla`.
- Troquei o mapa do Sudeste inteiro por um recorte da **Baixada Santista**, para deixar a construção dos tiles mais rápida.
- Criei o `routing-service` com Spring Boot (Web, Validation, Actuator e Lombok).
- Fiz a primeira requisição de rota de caminhão no endpoint `/route` do Valhalla.

### Conceitos que aprendi

- **`.pbf`**: o mapa "cru" do OpenStreetMap, baixado em formato compactado.
- **Tiles de roteamento**: o Valhalla converte o `.pbf` em tiles na primeira execução (por isso a primeira subida demora). Os tiles ficam salvos e as próximas subidas são rápidas. Por serem pesados, vão para o `.gitignore`.
- **`costing_options`**: é onde se informa ao Valhalla as medidas do caminhão (altura, largura e comprimento em metros; peso em toneladas).
- **Actuator**: expõe o `/actuator/health`, útil para saber se um serviço está de pé.

### A requisição que fiz

`POST http://localhost:8002/route`

```json
{
  "locations": [
    { "lat": -24.094649, "lon": -46.623519 },
    { "lat": -24.093534, "lon": -46.624558 }
  ],
  "costing": "truck",
  "costing_options": {
    "truck": {
      "height": 4.4,
      "width": 2.6,
      "length": 18.0,
      "weight": 40.0
    }
  },
  "units": "miles",
  "id": "my_work_route"
}
```

### O resultado

- A requisição retornou **200 OK** com a rota completa e todas as informações esperadas.
- **Problema:** a rota passa por uma ponte de aproximadamente 2 m de altura livre, e o Valhalla não a evitou, mesmo com um caminhão de 4,4 m.
- Local da ponte: _(anotar aqui as coordenadas ou o nome da ponte)_

### Por que isso aconteceu

O Valhalla só desvia de restrições que existem nos dados do OpenStreetMap (como a tag `maxheight`). Se essa ponte não tem a altura cadastrada no mapa, ele a trata como uma ponte comum. Não é um erro de código nem de configuração: é **falta de dado**.

### Por que isso é importante para o projeto

Esse é exatamente o problema que o projeto quer resolver: os dados de restrição para caminhões no Brasil são incompletos. A solução planejada é uma **camada própria de restrições**, alimentada por dados oficiais e por reports dos motoristas.

Esta requisição vira o **cenário de demonstração**:

1. Antes: a rota passa pela ponte de 2 m.
2. Depois (com a camada de restrições): a rota desvia dela.

