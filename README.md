# f
Fire game 
import pygame
import os
import sys

# Инициализация Pygame
pygame.init()

# Размер окна
WINDOW_SIZE = (800, 600)

# Цвета
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

# Создание окна
screen = pygame.display.set_mode(WINDOW_SIZE)

# Заголовок окна
pygame.display.set_caption("Two Player Platformer")

# Загрузка изображений
BACKGROUND_IMAGE = pygame.image.load(os.path.join("assets", "images", "background.jpg")).convert_alpha()
HERO1_IMAGE = pygame.image.load(os.path.join("assets", "images", "hero1.png")).convert_alpha()
HERO2_IMAGE = pygame.image.load(os.path.join("assets", "images", "hero2.png")).convert_alpha()
BLOCK_IMAGE = pygame.image.load(os.path.join("assets", "images", "block.png")).convert_alpha()
COIN_IMAGE = pygame.image.load(os.path.join("assets", "images", "coin.png")).convert_alpha()
FINISH_IMAGE = pygame.image.load(os.path.join("assets", "images", "finish.png")).convert_alpha()
GATE_IMAGE = pygame.image.load(os.path.join("assets", "images", "gate.png")).convert_alpha()
LEVER_IMAGE = pygame.image.load(os.path.join("assets", "images", "lever.png")).convert_alpha()
TRAP_IMAGE = pygame.image.load(os.path.join("assets", "images", "trap.png")).convert_alpha()
WATER_IMAGE = pygame.image.load(os.path.join("assets", "images", "water.png")).convert_alpha()

# Загрузка звуков
COIN_SOUND = pygame.mixer.Sound(os.path.join("assets", "sounds", "coin.wav"))
DEATH_SOUND = pygame.mixer.Sound(os.path.join("assets", "sounds", "death.wav"))
JUMP_SOUND = pygame.mixer.Sound(os.path.join("assets", "sounds", "jump.wav"))
VICTORY_SOUND = pygame.mixer.Sound(os.path.join("assets", "sounds", "victory.wav"))

# Класс игрока
class Player(pygame.sprite.Sprite):
    def __init__(self, x, y, image, up_key, down_key, left_key, right_key):
        super().__init__()

        # Параметры игрока
        self.x = x
        self.y = y
        self.vx = 0
        self.vy = 0
        self.image = image
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

        # Клавиши управления
        self.up_key = up_key
        self.down_key = down_key
        self.left_key = left_key
        self.right_key = right_key

        # Состояние игрока
        self.is_alive = True
        self.coins = 0
        self.has_key = False

    # Обновление игрока
    def update(self, blocks, coins, finish, gates, levers, traps, water):
        # Движение игрока
        self.vx = 0
        self.vy += 1
        keys = pygame.key.get_pressed()
        if keys[self.up_key]:
            if self.rect.bottom >= 600:
                self.vy = -20
                            JUMP_SOUND.play()
            self.vy = -20
        if keys[self.left_key]:
            self.vx = -5
        if keys[self.right_key]:
            self.vx = 5
        self.rect.x += self.vx
        self.rect.y += self.vy

        # Проверка столкновений с блоками
        for block in blocks:
            if self.rect.colliderect(block.rect):
                if self.vx > 0:
                    self.rect.right = block.rect.left
                elif self.vx < 0:
                    self.rect.left = block.rect.right
                if self.vy > 0:
                    self.rect.bottom = block.rect.top
                    self.vy = 0
                elif self.vy < 0:
                    self.rect.top = block.rect.bottom
                    self.vy = 0

        # Проверка столкновений с монетами
        for coin in coins:
            if self.rect.colliderect(coin.rect):
                coins.remove(coin)
                self.coins += 1
                COIN_SOUND.play()

        # Проверка столкновений с финишем
        if self.rect.colliderect(finish.rect):
            VICTORY_SOUND.play()
            pygame.time.delay(1000)
            pygame.quit()
            sys.exit()

        # Проверка столкновений с воротами
        for gate in gates:
            if self.rect.colliderect(gate.rect):
                if self.has_key:
                    gates.remove(gate)
                    self.has_key = False
                else:
                    if self.vx > 0:
                        self.rect.right = gate.rect.left
                    elif self.vx < 0:
                        self.rect.left = gate.rect.right
                    if self.vy > 0:
                        self.rect.bottom = gate.rect.top
                        self.vy = 0
                    elif self.vy < 0:
                        self.rect.top = gate.rect.bottom
                        self.vy = 0

        # Проверка столкновений с рычагами
        for lever in levers:
            if self.rect.colliderect(lever.rect):
                lever.toggle()

        # Проверка столкновений с ловушками
        for trap in traps:
            if self.rect.colliderect(trap.rect):
                DEATH_SOUND.play()
                pygame.time.delay(1000)
                pygame.quit()
                sys.exit()

        # Проверка столкновений с опасной жидкостью
        if self.rect.colliderect(water.rect):
            DEATH_SOUND.play()
            pygame.time.delay(1000)
            pygame.quit()
            sys.exit()

    # Рисование игрока
    def draw(self, surface):
        surface.blit(self.image, self.rect)

# Класс блока
class Block(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        super().__init__()

        # Параметры блока
        self.image = image
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

    # Рисование блока
    def draw(self, surface):
        surface.blit(self.image, self.rect)

# Класс монеты
class Coin(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        super().__init__()

                self.rect.x = x
        self.rect.y = y

    # Рисование монеты
    def draw(self, surface):
        surface.blit(self.image, self.rect)

# Класс финиша
class Finish(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        super().__init__()

        # Параметры финиша
        self.image = image
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

    # Рисование финиша
    def draw(self, surface):
        surface.blit(self.image, self.rect)

# Класс ворот
class Gate(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        super().__init__()

        # Параметры ворот
        self.image = image
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

    # Рисование ворот
    def draw(self, surface):
        surface.blit(self.image, self.rect)

# Класс рычага
class Lever(pygame.sprite.Sprite):
    def __init__(self, x, y, image_on, image_off, gates):
        super().__init__()

        # Параметры рычага
        self.image_on = image_on
        self.image_off = image_off
        self.image = self.image_off
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
        self.gates = gates

    # Изменение состояния рычага и ворот
    def toggle(self):
        self.image = self.image_on if self.image == self.image_off else self.image_off
        for gate in self.gates:
            gate.rect.y += 50 if self.image == self.image_on else -50

    # Рисование рычага
    def draw(self, surface):
        surface.blit(self.image, self.rect)

# Класс ловушки
class Trap(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        super().__init__()

        # Параметры ловушки
        self.image = image
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

    # Рисование ловушки
    def draw(self, surface):
        surface.blit(self.image, self.rect)

# Класс опасной жидкости
class Water(pygame.sprite.Sprite):
    def __init__(self, x, y, image):
        super().__init__()

        # Параметры опасной жидкости
        self.image = image
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

    # Рисование опасной жидкости
    def draw(self, surface):
        surface.blit(self.image, self.rect)

# Инициализация Pygame и создание окна
pygame.init()
pygame.mixer.init()
screen_width, screen_height = 800, 600
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption('Two Player Platformer')

# Загрузка изображений
PLAYER_1_IMAGE = pygame.image.load('player1.png').convert_alpha()
PLAYER_2_IMAGE = pygame.image.load('player2.png').convert_alpha()

class Obstacle:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.width = 50
        self.height = 50

    def draw(self, surface):
        pygame.draw.rect(surface, (255, 0, 0), (self.x, self.y, self.width, self.height))
class Gate:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.width = 50
        self.height = 100

    def draw(self, surface):
        pygame.draw.rect(surface, (0, 0, 255), (self.x, self.y, self.width, self.height))

class Liquid:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.width = 50
        self.height = 50

    def draw(self, surface):
        pygame.draw.rect(surface, (0, 255, 0), (self.x, self.y, self.width, self.height))

class Player:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.width = 50
        self.height = 50
        self.speed = 5

    def move(self, keys):
        if keys[pygame.K_LEFT]:
            self.x -= self.speed
        if keys[pygame.K_RIGHT]:
            self.x += self.speed
        if keys[pygame.K_UP]:
            self.y -= self.speed
        if keys[pygame.K_DOWN]:
            self.y += self.speed

    def draw(self, surface):
        pygame.draw.rect(surface, (255, 255, 255), (self.x, self.y, self.width,


def main():
    pygame.init()
    pygame.mixer.init()
    screen = pygame.display.set_mode((WIDTH, HEIGHT))
    pygame.display.set_caption("Co-op Game")
    clock = pygame.time.Clock()
    font = pygame.font.SysFont(None, 48)
    game_over_sound = pygame.mixer.Sound(os.path.join(SOUND_DIR, "game_over.wav"))

    player1 = Player("player1.png", "player1_walk.png", "player1_jump.png", "player1_fall.png", (50, 50), PLAYER_SPEED, PLAYER_JUMP_SPEED)
    player2 = Player("player2.png", "player2_walk.png", "player2_jump.png", "player2_fall.png", (WIDTH - 100, 50), PLAYER_SPEED, PLAYER_JUMP_SPEED)

    level_list = []
    level_list.append(Level("level1.txt", player1, player2))
    level_list.append(Level("level2.txt", player1, player2))
    current_level_no = 0
    current_level = level_list[current_level_no]
    active_sprite_list = pygame.sprite.Group()
    player1.level = current_level
    player2.level = current_level
    active_sprite_list.add(player1)
    active_sprite_list.add(player2)

    running = True
    game_over = False

    while running:
        if game_over:
            game_over_sound.play()
            text = font.render("Game Over", True, RED)
            text_rect = text.get_rect(center=(WIDTH//2, HEIGHT//2))
            screen.blit(text, text_rect)
            pygame.display.flip()
            pygame.time.wait(3000)
            running = False
        else:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_ESCAPE:
                        running = False
                    elif event.key == pygame.K_p:
                        pause(screen)

            keys = pygame.key.get_pressed()
            if keys[pygame.K_LEFT]:
                player1.go_left()
            elif keys[pygame.K_RIGHT]:
                player1.go_right()
            elif keys[pygame.K_UP]:
                player1.jump()
            elif keys[pygame.K_DOWN]:
                player1.crouch()
            else:
                player1.stop()

            if keys[pygame.K_a]:
                player2.go_left()
            elif keys[pygame.K_d]:
                player2.go_right()
            elif keys[pygame.K_w]:
                player2.jump()
            elif keys[pygame.K_s]:
                player2.crouch()
            else:
                player2.stop()

            active_sprite_list.update()

            if player1.rect.right >= WIDTH:
                player1.rect.right = WIDTH
            elif player1.rect.left <= 0:
                player1.rect.left = 0

            if player2.rect.right >= WIDTH:
                player2.rect.right = WIDTH
            elif player2.rect.left <= 0:
                player2.rect.left = 0

            current_level.draw(screen)
            active_sprite_list.draw(screen)
            pygame.display.flip()

            if player1.is_dead() or player2.is_dead():
                game_over = True

            if current_level.is_complete():
                current_level_no += 1
                if current_level_no == len(level_list):
                    running = False
                else:
                    current_level = level_list[current_level_no]
                    player1.level = current_level
                    player2.level = current_level

        clock.tick(FPS)

    pygame.quit()

if __name__ == "__main__":
    main()



