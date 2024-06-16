import pygame
import sys

# Инициализация Pygame
pygame.init()

# Размеры окна
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600

# Цвета
WHITE = (255, 255, 255)

# Создание окна игры
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption('Стартап: начальное меню')

# Загрузка изображений для начального меню
background_menu_orig = pygame.image.load('../pythonProject1/assets/background_menu.png').convert()
button_start_orig = pygame.image.load('../pythonProject1/assets/button_start.png').convert_alpha()

# Установка новых размеров фона и кнопки старта
background_menu = pygame.transform.scale(background_menu_orig, (SCREEN_WIDTH, SCREEN_HEIGHT))
button_start = pygame.transform.scale(button_start_orig, (200, 50))

# Получение прямоугольника (rect) для кнопки "Старт"
button_rect = button_start.get_rect(center=(SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2 + 20))

# Загрузка изображений для игрового режима и уменьшение их размеров
game_background = pygame.transform.scale(pygame.image.load('../pythonProject1/assets/game_background.png').convert(), (SCREEN_WIDTH, SCREEN_HEIGHT))
startup_default = pygame.transform.scale(pygame.image.load('../pythonProject1/assets/startup.png').convert_alpha(), (100, 150))
startup_jump = pygame.transform.scale(pygame.image.load('../pythonProject1/assets/startup_jump.png').convert_alpha(), (100, 150))
startup_fail = pygame.transform.scale(pygame.image.load('../pythonProject1/assets/startup_fail.png').convert_alpha(), (100, 150))
demon_investor_before = pygame.transform.scale(pygame.image.load(
    '../pythonProject1/assets/demon_investor_before.png').convert_alpha(), (100, 150))
demon_investor_after = pygame.transform.scale(pygame.image.load(
    '../pythonProject1/assets/demon_investor_after.png').convert_alpha(), (100, 150))

# Получение прямоугольников (rect) для персонажей
startup_rect = startup_default.get_rect(bottomleft=(100, SCREEN_HEIGHT - 50))
demon_investor_rect = demon_investor_before.get_rect(bottomleft=(SCREEN_WIDTH, SCREEN_HEIGHT - 50))

# Переменные для механики игры
is_jumping = False
is_falling = False
is_game_started = False
is_game_over = False
jump_velocity = -25  # Увеличенная начальная скорость прыжка
gravity = 0.7  # Уменьшенная гравитация
horizontal_speed = 5  # Новая переменная для горизонтальной скорости в прыжке
score = 0
demon_investor_speed = 3
game_over_time = 2000  # 2 секунды

# Установка начального состояния изображения демона-инвестора
demon_investor_image = demon_investor_before

# Переменная для хранения начальной позиции демона-инвестора
demon_investor_initial_pos = demon_investor_rect.bottomleft

# Переменная для хранения начальной позиции стартапера
startup_initial_pos = startup_rect.bottomleft

# Функция для возврата к начальному меню
def show_menu():
    global is_game_started, is_game_over, score, startup_rect, demon_investor_rect, jump_velocity, demon_investor_image
    is_game_started = False
    is_game_over = False
    score = 0
    startup_rect.bottomleft = startup_initial_pos
    demon_investor_rect.bottomleft = demon_investor_initial_pos
    jump_velocity = -25
    demon_investor_image = demon_investor_before

# Основной цикл игры
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        elif event.type == pygame.MOUSEBUTTONDOWN and not is_game_started:
            # Проверка нажатия кнопки "Старт"
            if button_rect.collidepoint(event.pos):
                is_game_started = True
                is_game_over = False
                score = 0
                startup_rect.bottomleft = (100, SCREEN_HEIGHT - 50)
                demon_investor_rect.bottomleft = demon_investor_initial_pos
                demon_investor_image = demon_investor_before

        elif event.type == pygame.KEYDOWN and is_game_started and not is_game_over:
            if event.key == pygame.K_SPACE and not is_jumping and not is_falling:
                is_jumping = True
                jump_velocity = -25  # Увеличенная начальная скорость прыжка

    # Логика прыжка стартапера
    if is_jumping:
        startup_rect.x += horizontal_speed  # Горизонтальное смещение вправо
        startup_rect.y += jump_velocity
        jump_velocity += gravity
        if jump_velocity >= 0:
            is_jumping = False
            is_falling = True

    if is_falling:
        startup_rect.x += horizontal_speed  # Горизонтальное смещение вправо
        startup_rect.y += jump_velocity
        jump_velocity += gravity
        if startup_rect.bottom >= SCREEN_HEIGHT - 50:
            startup_rect.bottom = SCREEN_HEIGHT - 50
            is_falling = False
            if startup_rect.colliderect(demon_investor_rect):
                is_game_over = True

    # Движение демона-инвестора
    if is_game_started and not is_game_over:
        demon_investor_rect.x -= demon_investor_speed
        if demon_investor_rect.right <= 0:
            demon_investor_rect.bottomleft = demon_investor_initial_pos
            score += 1

    # Проверка на столкновение с демоном-инвестором
    if startup_rect.colliderect(demon_investor_rect) and not is_game_over:
        is_game_over = True
        game_over_start_time = pygame.time.get_ticks()

    # Отрисовка элементов на экране
    if not is_game_started:
        screen.blit(background_menu, (0, 0))
        screen.blit(button_start, button_rect.topleft)
    else:
        screen.blit(game_background, (0, 0))
        if is_game_over:
            screen.blit(startup_fail, startup_rect.topleft)
            if pygame.time.get_ticks() - game_over_start_time >= game_over_time:
                show_menu()
        else:
            # Проверяем, что стартапер не находится в состоянии прыжка или падения, прежде чем его отрисовать
            if not is_jumping and not is_falling:
                screen.blit(startup_default, startup_rect.topleft)
            elif is_jumping:
                screen.blit(startup_jump, startup_rect.topleft)
            elif is_falling:
                screen.blit(startup_jump, startup_rect.topleft)

            screen.blit(demon_investor_image, demon_investor_rect.topleft)

        # Отрисовка счета
        font = pygame.font.SysFont(None, 55)
        score_text = font.render(f'Score: {score}', True, WHITE)
        screen.blit(score_text, (10, 10))

    # Обновление экрана
    pygame.display.flip()
