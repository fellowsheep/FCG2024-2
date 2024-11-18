# Introdução aos Sprites

![image](https://github.com/user-attachments/assets/dba6f45e-2d4f-483b-9623-35e719154321)
Fonte: autoral

No contexto do desenvolvimento de jogos, **sprites** são imagens que são mapeadas como textura em uma geometria simples (geralmente retangular), utilizadas para representar personagens, objetos ou elementos visuais dentro de uma cena. Um sprite é desenhado em um plano cartesiano e pode ser manipulado para aplicar transformações como translação, rotação e escala.
Código-fonte de apoio: link aqui

---

## Estrutura de Dados para Sprites

Os sprites em nosso exemplo são representados por uma `struct Sprite`, que contém os seguintes campos:

- **VAO (Vertex Array Object)**: O identificador do buffer de geometria, que representa um quadrilátero centrado na origem do sistema cartesiano, com suas coordenadas locais e dimensões 1x1. Nesse VAO temos os ponteiros para os atributos do vértice **posição** e **coordenadas de textura**.

![image](https://github.com/user-attachments/assets/deba4042-adbc-4ea0-859a-848704c809ee)
Fonte: autora. Sprite obtida do itch.io (https://pipoya.itch.io/pipoya-free-rpg-character-sprites-32x32)



- **texID**: O identificador da textura associada ao sprite, que conectamos antes de fazer a chamada de desenho.
- **Transformações**: Variáveis para armazenar:
  - Posição (`vec3 pos`)
  - Escala (`vec3 dimensions`)
  - Rotação (`float angle`) - consideramos, neste caso, que todas as rotações ocorrerão no eixo de profundidade (eixo `z`)

### Código da Estrutura
\```cpp
struct Sprite 
{
	GLfloat VAO;        // ID do buffer de geometria
	GLfloat texID;      // ID da textura
	vec3 pos;           // Posição do sprite
	vec3 dimensions;    // Dimensões (escala) do sprite
	float angle;        // Rotação
};
\```

---

## Mapeamento de Coordenadas

Para facilitar o posicionamento e a escala dos sprites, utilizamos uma projeção ortográfica que mapeia o plano `xy` diretamente para a viewport. Dessa forma, cada unidade de mundo é equivalente a 1 pixel na tela.

Por exemplo, para uma janela com dimensões **800x600**, a matriz de projeção pode ser definida assim:

\```cpp
glm::mat4 projection = glm::ortho(0.0f, 800.0f, 0.0f, 600.0f, -1.0f, 1.0f);
\```

Isso permite posicionar e escalar os sprites com facilidade. Um sprite no centro da tela pode ser posicionado aplicando uma translação de `(400, 300)`.

---

## Recursos Úteis

- **Repositório de Texturas**: Para encontrar texturas gratuitas para seus sprites, você pode visitar [CraftPix](https://craftpix.net/freebies/).

Para mais exemplos e explicações detalhadas, consulte o [diretório do projeto](https://github.com/fellowsheep/FCG2024-2/tree/main/HelloTriangle%20-%20Sprites).
