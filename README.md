aqui ta a linha de codigo


import pygame
import random
import sys
import os

# Configuração para executável
if getattr(sys, 'frozen', False):
    # Se estiver rodando como executável
    os.chdir(sys._MEIPASS)

# Inicialização
pygame.init()

# Configurações da tela
LARGURA = 400
ALTURA = 600
TELA = pygame.display.set_mode((LARGURA, ALTURA))
pygame.display.set_caption("Flappy Bird")
CLOCK = pygame.time.Clock()

# Cores
BRANCO = (255, 255, 255)
PRETO = (0, 0, 0)
VERDE = (0, 200, 0)
AZUL = (0, 150, 255)
AMARELO = (255, 255, 0)
VERMELHO = (255, 0, 0)

# Variáveis do jogo
GRAVIDADE = 0.5
PULO = -8
VELOCIDADE_CANO = 3
LARGURA_CANO = 70
ESPACO_ENTRE_CANOS = 180


class Passaro:
    def __init__(self):
        self.x = 100
        self.y = ALTURA // 2
        self.velocidade = 0
        self.tamanho = 20
        self.angulo = 0

    def pular(self):
        self.velocidade = PULO
        self.angulo = -30

    def atualizar(self):
        self.velocidade += GRAVIDADE
        self.y += self.velocidade

        # Ajustar ângulo baseado na velocidade
        if self.velocidade < 0:
            self.angulo = max(-30, self.angulo - 3)
        else:
            self.angulo = min(90, self.angulo + 3)

    def desenhar(self):
        # Desenhar corpo do pássaro
        pygame.draw.circle(TELA, AMARELO, (int(self.x), int(self.y)), self.tamanho)
        # Olho
        pygame.draw.circle(TELA, PRETO, (int(self.x + 8), int(self.y - 5)), 3)
        # Bico
        pygame.draw.polygon(TELA, (255, 140, 0), [
            (int(self.x + 15), int(self.y)),
            (int(self.x + 25), int(self.y)),
            (int(self.x + 20), int(self.y - 5))
        ])

    def get_rect(self):
        return pygame.Rect(self.x - self.tamanho, self.y - self.tamanho,
                           self.tamanho * 2, self.tamanho * 2)


class Cano:
    def __init__(self, x):
        self.x = x
        self.altura = random.randint(100, ALTURA - ESPACO_ENTRE_CANOS - 100)
        self.passou = False

    def atualizar(self):
        self.x -= VELOCIDADE_CANO

    def desenhar(self):
        # Cano de cima
        pygame.draw.rect(TELA, VERDE,
                         (self.x, 0, LARGURA_CANO, self.altura))
        pygame.draw.rect(TELA, (0, 150, 0),
                         (self.x - 10, self.altura - 40, LARGURA_CANO + 20, 40))

        # Cano de baixo
        altura_baixo = self.altura + ESPACO_ENTRE_CANOS
        pygame.draw.rect(TELA, VERDE,
                         (self.x, altura_baixo, LARGURA_CANO, ALTURA - altura_baixo))
        pygame.draw.rect(TELA, (0, 150, 0),
                         (self.x - 10, altura_baixo, LARGURA_CANO + 20, 40))

    def get_rect_cima(self):
        return pygame.Rect(self.x, 0, LARGURA_CANO, self.altura)

    def get_rect_baixo(self):
        return pygame.Rect(self.x, self.altura + ESPACO_ENTRE_CANOS,
                           LARGURA_CANO, ALTURA - (self.altura + ESPACO_ENTRE_CANOS))


def mostrar_texto(texto, tamanho, cor, x, y, centralizado=False):
    fonte = pygame.font.Font(None, tamanho)
    superficie = fonte.render(texto, True, cor)
    if centralizado:
        x = LARGURA // 2 - superficie.get_width() // 2
    TELA.blit(superficie, (x, y))


def desenhar_chao():
    pygame.draw.rect(TELA, (100, 100, 100), (0, ALTURA - 50, LARGURA, 50))
    pygame.draw.rect(TELA, (80, 80, 80), (0, ALTURA - 55, LARGURA, 5))
    for i in range(0, LARGURA, 50):
        pygame.draw.rect(TELA, (200, 200, 200), (i + (pygame.time.get_ticks() // 10 % 50), ALTURA - 45, 30, 10))


def jogo():
    passaro = Passaro()
    canos = []
    pontuacao = 0
    jogo_ativo = True
    tempo_ultimo_cano = pygame.time.get_ticks()

    while jogo_ativo:
        delta_time = CLOCK.tick(60)

        # Eventos
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                return -1
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_SPACE or evento.key == pygame.K_UP:
                    passaro.pular()
                if evento.key == pygame.K_ESCAPE:
                    return -1

        # Atualizar pássaro
        passaro.atualizar()

        # Verificar colisão com bordas
        if passaro.y <= 0 or passaro.y >= ALTURA - 50:
            jogo_ativo = False

        # Adicionar novos canos
        agora = pygame.time.get_ticks()
        if agora - tempo_ultimo_cano > 1500:
            canos.append(Cano(LARGURA))
            tempo_ultimo_cano = agora

        # Atualizar e verificar canos
        for cano in canos[:]:
            cano.atualizar()

            # Verificar colisão
            if passaro.get_rect().colliderect(cano.get_rect_cima()) or \
                    passaro.get_rect().colliderect(cano.get_rect_baixo()):
                jogo_ativo = False

            # Remover canos fora da tela
            if cano.x + LARGURA_CANO < 0:
                canos.remove(cano)

            # Aumentar pontuação
            if not cano.passou and cano.x + LARGURA_CANO < passaro.x:
                cano.passou = True
                pontuacao += 1

        # Desenhar
        TELA.fill(AZUL)

        # Desenhar nuvens (efeito)
        for i in range(3):
            pygame.draw.ellipse(TELA, (255, 255, 255, 100),
                                (50 + i * 100 + (pygame.time.get_ticks() // 100 % 400),
                                 50 + i * 80, 60, 40))

        passaro.desenhar()
        for cano in canos:
            cano.desenhar()

        desenhar_chao()

        # Mostrar pontuação
        mostrar_texto(str(pontuacao), 50, BRANCO, LARGURA // 2, 50, centralizado=True)

        pygame.display.update()

    return pontuacao


def menu():
    while True:
        TELA.fill(AZUL)

        # Título animado
        titulo_y = 150 + (pygame.time.get_ticks() // 200 % 10 - 5)
        mostrar_texto("FLAPPY BIRD", 50, BRANCO, LARGURA // 2, titulo_y, centralizado=True)

        mostrar_texto("Pressione ESPAÇO para jogar", 25, BRANCO, LARGURA // 2, 300, centralizado=True)
        mostrar_texto("Pressione ESC para sair", 25, BRANCO, LARGURA // 2, 350, centralizado=True)

        # Desenhar pássaro no menu
        pygame.draw.circle(TELA, AMARELO, (LARGURA // 2, 450), 15)
        pygame.draw.circle(TELA, PRETO, (LARGURA // 2 + 8, 445), 2)

        pygame.display.update()

        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_SPACE:
                    pontuacao = jogo()
                    if pontuacao == -1:
                        pygame.quit()
                        sys.exit()
                    mostrar_game_over(pontuacao)
                if evento.key == pygame.K_ESCAPE:
                    pygame.quit()
                    sys.exit()


def mostrar_game_over(pontuacao):
    while True:
        TELA.fill(AZUL)
        mostrar_texto("GAME OVER", 50, VERMELHO, LARGURA // 2, 150, centralizado=True)
        mostrar_texto(f"Pontuação: {pontuacao}", 30, BRANCO, LARGURA // 2, 250, centralizado=True)

        if pontuacao >= 10:
            mostrar_texto("Parabéns! Ótima pontuação!", 20, AMARELO, LARGURA // 2, 320, centralizado=True)

        mostrar_texto("Pressione ESPAÇO para jogar novamente", 25, BRANCO, LARGURA // 2, 400, centralizado=True)
        mostrar_texto("Pressione ESC para sair", 25, BRANCO, LARGURA // 2, 450, centralizado=True)

        pygame.display.update()

        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_SPACE:
                    return
                if evento.key == pygame.K_ESCAPE:
                    pygame.quit()
                    sys.exit()


if __name__ == "__main__":
    menu()
