# Introdução aos Sprites

> ⚠ **Aviso:** Este material é complementar e se refere ao código disponível no seguinte link:  
[Sprites.cpp - Código Fonte de Apoio](https://github.com/fellowsheep/FCG2024-2/blob/main/HelloTriangle%20-%20Sprites/Sprites.cpp)

<p align="center">
<img src = "https://github.com/user-attachments/assets/dba6f45e-2d4f-483b-9623-35e719154321" alt="Exemplo cena com sprites" />
</p> <p align = "center"><em>Exemplo de cena composta de vários sprites. Fonte: autoral Assets: Freepik</em></p>

No contexto do desenvolvimento de jogos, **sprites** são imagens que são mapeadas como textura em uma geometria simples (geralmente retangular), utilizadas para representar personagens, objetos ou elementos visuais dentro de uma cena. Um sprite é desenhado em um plano cartesiano e pode ser manipulado para aplicar transformações como translação, rotação e escala.

---

## Estrutura de Dados para Sprites

Os sprites em nosso exemplo são representados por uma `struct Sprite`, que contém os seguintes campos:

- **VAO (Vertex Array Object)**: O identificador do buffer de geometria, que representa um quadrilátero centrado na origem do sistema cartesiano, com suas coordenadas locais e dimensões 1x1. Nesse VAO temos os ponteiros para os atributos do vértice **posição** e **coordenadas de textura**.

<p align="center">
<img src = "https://github.com/user-attachments/assets/deba4042-adbc-4ea0-859a-848704c809ee" alt="Mapeamento entre sistemas de coordenadas em um sprite." width="800" />
</p> <p align = "center"><em>Mapeamento entre sistemas de coordenadas em um sprite. Fonte: autoral Assets: https://pipoya.itch.io/pipoya-free-rpg-character-sprites-32x32</em></p>

- **texID**: O identificador da textura associada ao sprite, que conectamos antes de fazer a chamada de desenho.
- **Transformações**: Variáveis para armazenar:
  - Posição (`vec3 pos`)
  - Escala (`vec3 dimensions`)
  - Rotação (`float angle`) - consideramos, neste caso, que todas as rotações ocorrerão no eixo de profundidade (eixo `z`)

### Código da Estrutura
```cpp
struct Sprite 
{
	GLfloat VAO;        // ID do buffer de geometria
	GLfloat texID;      // ID da textura
	vec3 pos;           // Posição do sprite
	vec3 dimensions;    // Dimensões (escala) do sprite
	float angle;        // Rotação
};
```

---

## Mapeamento de Coordenadas

Para facilitar o posicionamento e a escala dos sprites, utilizamos uma projeção ortográfica que mapeia o plano `xy` diretamente para a viewport. Dessa forma, cada unidade de mundo é equivalente a 1 pixel na tela.

Por exemplo, para uma janela com dimensões **800x600**, a matriz de projeção pode ser definida assim:

```cpp
glm::mat4 projection = glm::ortho(0.0f, 800.0f, 0.0f, 600.0f, -1.0f, 1.0f);
```

Isso permite posicionar e escalar os sprites com facilidade. Um sprite no centro da tela pode ser posicionado aplicando uma translação de `(400, 300)`.

---

## Funções Importantes

### Função `initializeSprite`

Essa função inicializa um sprite conforme o mapeamento mostrado na seção anterior (considerando uma imagem comum, sem spritesheet), criando seu VAO e VBO. Os vértices do sprite são definidos para formar um quadrilátero centrado na origem. Até aqui, nossos sprites são equivalentes a qualquer quadrilátero com uma imagem inteira mapeada para ele. Por simplicidade, optou-se criar apenas um VBO contendo os dois triângulos sem o compartilhamento dos vértices, mas fica como desafio aos estudantes utilizar o buffer EBO (Element Buffer Object) ou ainda explorar a primitiva de desenho `GL_TRIANGLE_STRIP` 😎

```cpp
Sprite initializeSprite(GLuint texID, vec3 dimensions, vec3 position, float angle)
{
	Sprite sprite;

	sprite.texID = texID;
	sprite.dimensions.x = dimensions.x;
	sprite.dimensions.y = dimensions.y;
	sprite.pos = position;
	sprite.angle = angle;

	GLfloat vertices[] = {
		// x    y    z   s    t 
		// T0
		-0.5,  0.5, 0.0, 0.0, 1.0,  // v0
		-0.5, -0.5, 0.0, 0.0, 0.0,  // v1
		 0.5,  0.5, 0.0, 1.0, 1.0,  // v2
		// T1
		-0.5, -0.5, 0.0, 0.0, 0.0,  // v1
		 0.5,  0.5, 0.0, 1.0, 1.0,  // v2
		 0.5, -0.5, 0.0, 1.0, 0.0   // v3
	};

	GLuint VBO, VAO;
	// Geração do identificador do VBO
	glGenBuffers(1, &VBO);
	// Faz a conexão (vincula) do buffer como um buffer de array
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	// Envia os dados do array de floats para o buffer da OpenGl
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

	// Geração do identificador do VAO (Vertex Array Object)
	glGenVertexArrays(1, &VAO);
	// Vincula (bind) o VAO primeiro, e em seguida  conecta e seta o(s) buffer(s) de vértices
	// e os ponteiros para os atributos
	glBindVertexArray(VAO);
	// Para cada atributo do vertice, criamos um "AttribPointer" (ponteiro para o atributo)
	
	//Atributo posição - coord x, y, z - 3 valores
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 5 * sizeof(GLfloat), (GLvoid *)0);
	glEnableVertexAttribArray(0);

	//Atributo coordenada de textura - coord s, t - 2 valores
	glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, 5 * sizeof(GLfloat), (GLvoid *)(3* sizeof(GLfloat)));
	glEnableVertexAttribArray(1);

	// Observe que isso é permitido, a chamada para glVertexAttribPointer registrou o VBO como o objeto de buffer de vértice
	// atualmente vinculado - para que depois possamos desvincular com segurança
	glBindBuffer(GL_ARRAY_BUFFER, 0);

	// Desvincula o VAO (é uma boa prática desvincular qualquer buffer ou array para evitar bugs medonhos)
	glBindVertexArray(0);

	sprite.VAO = VAO;

    	return sprite;
}
```

### Função `drawSprite`

Essa função desenha um sprite utilizando shaders para aplicar as transformações de translação, escala e rotação.

```cpp
void drawSprite(GLuint shaderID, Sprite &sprite)
{
	glBindVertexArray(sprite.VAO); // Conectando ao buffer de geometria
	glBindTexture(GL_TEXTURE_2D, sprite.texID); //conectando com o buffer de textura que será usado no draw

	// Matriz de modelo: transformações na geometria (objeto)
	mat4 model = mat4(1); // matriz identidade
	// Translação
	model = translate(model, sprite.pos);
	// Rotação
	model = rotate(model, radians(sprite.angle), vec3(0.0,0.0,1.0));
	// Escala
	model = scale(model, sprite.dimensions);
	
	glUniformMatrix4fv(glGetUniformLocation(shaderID, "model"), 1, GL_FALSE, value_ptr(model));

	glDrawArrays(GL_TRIANGLES, 0, 6);

	glBindVertexArray(0); // Desconectando ao buffer de geometria
	glBindTexture(GL_TEXTURE_2D, 0); // Desconectando com o buffer de textura 

}
```
## Exemplo de criação de um Sprite

Considere que a sprite de nossa personagem é uma imagem chamada `Female 23.png` e que queremos mantê-la com seu tamanho original (32 x 23 pixels), uma vez que nossa janela do mundo (mapeada pela projeção ortográfica) está com o mesmo tamanho de unidades de pixels da tela (nossa viewport ocupa o tamanho da janela 800 x 600 pixels). Dessa forma, temos um mapeamento 1 x 1 (coordenada de mundo/coordenada de tela), e isso facilita muito dimensionarmos e posicionarmos nossas sprites no mundo. O código abaixo mostra como fazemos a criação de uma sprite para a personagem.

```cpp
//Criação dos sprites - objetos da cena
Sprite  character;
int imgWidth, imgHeight, texID;

// Carregando uma textura do personagem e armazenando seu id
texID = loadTexture("../Textures/Characters/Female 23.png",imgWidth,imgHeight);
character = initializeSprite(texID, vec3(imgWidth,imgHeight,1.0),vec3(400,100,0));
```
A função loadTexture (estudada nas aulas de mapeamento de textura), além de retornar o ID do buffer de textura que a OpenGL criou, "retorna" (pela passagem de parâmetros por referência, que permite que as alterações feitas nos mesmos se mantenham) as dimensões (em pixels) da imagem lida. Estas dimensões podem ser usadas diretamente como fator de escala para nosso sprite. 

Já a posição, que foi colocada de forma **hardcoded** neste exemplo: `vec3(400,100,0)`, corresponde exatamente à contagem de pixels em `x` e `y` da janela da aplicação (que foi mapeada como uma única _viewport_). Lembrando que neste nosso mapeamento, a `origem` do sistema de coordenadas de tela está no canto inferior esquerdo da tela.


## Recursos Úteis

- **Repositório de Texturas**: Para encontrar texturas gratuitas para seus sprites, você pode visitar [CraftPix](https://craftpix.net/freebies/).

Para mais exemplos e explicações detalhadas, consulte o [diretório do projeto](https://github.com/fellowsheep/FCG2024-2/tree/main/HelloTriangle%20-%20Sprites).
