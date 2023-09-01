import pygame
import sys
import random
import math

# Inicialização do Pygame
pygame.init()

# Configurações da tela
largura = 800
altura = 600
tela = pygame.display.set_mode((largura, altura))
pygame.display.set_caption("Jogo Completo")

# Cores
branco = (255, 255, 255)

# Sons
som_colisao = pygame.mixer.Sound("colisao_asteroide.wav")
som_derrota = pygame.mixer.Sound("som_derrota.wav")

# Fonte
fonte = pygame.font.Font(None, 36)

# Personagem (astronauta)
astronauta = pygame.image.load("astronauta.png")
astronauta = pygame.transform.scale(astronauta, (50, 50))
astronauta_rect = astronauta.get_rect()
astronauta_rect.center = (largura // 2, altura // 2)

# Inimigos
inimigos = []
for _ in range(5):
    inimigo = pygame.image.load("inimigo.png")
    inimigo = pygame.transform.scale(inimigo, (50, 50))
    inimigo_rect = inimigo.get_rect()
    inimigo_rect.x = random.randint(0, largura - 50)
    inimigo_rect.y = random.randint(0, altura - 50)
    inimigos.append(inimigo_rect)

# Velocidade do astronauta
velocidade_astronauta = 5

# Velocidade dos inimigos
velocidade_inimigo = 3

# Habilidade especial
habilidade_especial_ativa = False
habilidade_especial_cooldown = 5000  # Tempo de recarga em milissegundos (5 segundos)
habilidade_especial_tempo_uso = 0  # Tempo atual de uso da habilidade especial

# Experiência e nível
experiencia = 0
nivel = 1

# Botão "Subir de Nível"
botao_subir_nivel = pygame.image.load("botao_subir_nivel.png")
botao_subir_nivel_rect = botao_subir_nivel.get_rect()
posicao_botao_subir_nivel = (largura // 2 - botao_subir_nivel.get_width() // 2, altura // 2 + 50)

# Loop do menu
menu_ativo = True
while menu_ativo:
    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if evento.type == pygame.MOUSEBUTTONDOWN:
            if botao_subir_nivel_rect.collidepoint(evento.pos):
                menu_ativo = False  # Iniciar o jogo
                pygame.mixer.music.load("musica_fundo.wav")
                pygame.mixer.music.play(-1)

    # Desenhar o menu
    tela.fill(branco)

    texto_stats = fonte.render(f"Força: {velocidade_astronauta}  Agilidade: {velocidade_inimigo}", True, (0, 0, 0))
    tela.blit(texto_stats, (10, 10))

    texto_nivel = fonte.render(f"Nível: {nivel}", True, (0, 0, 0))
    tela.blit(texto_nivel, (10, 50))

    tela.blit(botao_subir_nivel, posicao_botao_subir_nivel)

    pygame.display.flip()

# Loop principal
while True:
    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

    # Movimento do astronauta
    teclas = pygame.key.get_pressed()
    if teclas[pygame.K_LEFT]:
        astronauta_rect.x -= velocidade_astronauta
    if teclas[pygame.K_RIGHT]:
        astronauta_rect.x += velocidade_astronauta
    if teclas[pygame.K_UP]:
        astronauta_rect.y -= velocidade_astronauta
    if teclas[pygame.K_DOWN]:
        astronauta_rect.y += velocidade_astronauta

    # Ativa a habilidade especial
    if habilidade_especial_ativa:
        # Limpa todos os inimigos
        inimigos.clear()
        # Verifica o tempo de uso da habilidade especial
        tempo_atual = pygame.time.get_ticks()
        if tempo_atual - habilidade_especial_tempo_uso >= habilidade_especial_cooldown:
            habilidade_especial_ativa = False

    # Movimento dos inimigos (IA simples - persegue o jogador)
    for inimigo in inimigos:
        dx = astronauta_rect.x - inimigo.x
        dy = astronauta_rect.y - inimigo.y
        distancia = math.sqrt(dx**2 + dy**2)

        if distancia != 0:
            dx = dx / distancia
            dy = dy / distancia

        inimigo.x += dx * velocidade_inimigo
        inimigo.y += dy * velocidade_inimigo

    # Detecção de colisões com inimigos
    for inimigo in inimigos:
        if astronauta_rect.colliderect(inimigo):
            som_derrota.play()
            print("Você foi pego por um inimigo! Game Over.")
            pygame.quit()
            sys.exit()

    # Detecção de colisão com experiência
    for inimigo in inimigos:
        if astronauta_rect.colliderect(inimigo):
            experiencia += random.randint(10, 50)

    # Verifica se o jogador subiu de nível
    if experiencia >= nivel * 100:
        nivel += 1
        velocidade_astronauta += 2
        velocidade_inimigo += 2

    # Desenhar a tela
    tela.fill(branco)

    # Desenhar o astronauta na tela
    tela.blit(astronauta, astronauta_rect)

    # Desenhar os inimigos na tela
    for inimigo in inimigos:
        tela.blit(inimigo, inimigo)

    # Exibir informações na tela
    texto_experiencia = fonte.render(f"Experiência: {experiencia}", True, (0, 0, 0))
    tela.blit(texto_experiencia, (10, 10))

    texto_nivel = fonte.render(f"Nível: {nivel}", True, (0, 0, 0))
    tela.blit(texto_nivel, (10, 50))

    pygame.display.flip()
