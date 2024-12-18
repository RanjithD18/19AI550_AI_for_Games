
# **Cosmic Heat: A 2D Space Shooter Game**
### DATE: 25-10-2024
### REGISTER NUMBER : 212221240044
## **Aim**
The aim of this project is to develop a 2D space shooter game, **Cosmic Heat**, using Python and Pygame. This game combines classic arcade shooting mechanics with modern AI techniques to create an engaging gameplay experience. Players control a spaceship using either the keyboard or a joystick to fight through waves of enemies and challenging boss battles. The AI techniques are implemented to enhance enemy behavior and make each encounter dynamic and challenging.

### Algorithm

1. **Start the Program**
   - Initialize the Python environment.
   - Import required libraries, including `pygame` for game graphics and controls, and other necessary AI and math libraries.

2. **Initialize Pygame and Game Settings**
   - Set up the Pygame display window with a specific width and height.
   - Define variables for game assets such as background, player ship, enemies, bosses, and sounds.
   - Define constants for game parameters such as screen dimensions, player speed, bullet speed, enemy speed, etc.

3. **Define Player and Enemy Classes**
   - Create a `Player` class with attributes for position, speed, health, and methods for movement and shooting.
   - Create an `Enemy` class to handle enemy attributes and actions, using AI techniques to define behavior.

4. **Implement AI Algorithms for Enemy Behavior**
   - **Finite State Machines (FSMs)**: Set up states like patrolling, attacking, and retreating, and specify transitions between these states based on player proximity and health.
   - **Pathfinding (A* Algorithm)**: Implement A* or similar algorithm for enemy movement, especially for bosses, allowing them to navigate toward the player while avoiding obstacles.
   - **Rule-Based Systems**: Define rules for simple enemy actions like shooting when within range, moving in specific patterns, or idling.
   - **Dynamic Difficulty Adjustment (DDA)**: Set up a mechanism to track player performance and dynamically adjust spawn rates and enemy behavior accordingly.

5. **Game Loop**
   - **Start the Game Loop**: Begin a continuous game loop to update gameplay elements.
   - **Check for Player Input**: Capture player keyboard or joystick input for moving the ship, shooting, pausing, or exiting the game.
   - **Update Player Position**: Adjust player position based on input.
   - **Update Enemy and Boss Behavior**: Use AI algorithms to change enemy positions, select targets, and update attack patterns.
   - **Collision Detection**: Check for collisions between player bullets and enemies, as well as enemy bullets and the player. If a collision occurs, adjust health or remove the enemy.
   - **Adjust Difficulty Dynamically**: Monitor player performance and adjust spawn rates, enemy health, or speed to match the player's skill level.
   - **Draw Elements**: Render the background, player, enemies, bullets, and other UI elements onto the screen.
   - **Update Display**: Refresh the Pygame display with updated game state visuals.

6. **Check for Game Over Condition**
   - If the player's health reaches zero, end the game.
   - Display a game-over message and offer an option to restart or exit.

7. **Quit the Game**
   - Stop the game loop and close the Pygame window upon exit.
### Program
### Main.py
```
import sys

import pygame
import random

from controls import move_player, move_player_with_joystick
from classes.constants import WIDTH, HEIGHT, FPS, SHOOT_DELAY
from functions import show_game_over, music_background
from menu import show_menu

from classes.player import Player
from classes.bullets import Bullet
from classes.refill import BulletRefill, HealthRefill, DoubleRefill, ExtraScore
from classes.meteors import Meteors, Meteors2, BlackHole
from classes.explosions import Explosion, Explosion2
from classes.enemies import Enemy1, Enemy2
from classes.bosses import Boss1, Boss2, Boss3


pygame.init()
music_background()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
surface = pygame.Surface((WIDTH, HEIGHT))
pygame.display.set_caption("Cosmic Heat")
clock = pygame.time.Clock()


explosions = pygame.sprite.Group()
explosions2 = pygame.sprite.Group()
bullets = pygame.sprite.Group()
enemy1_group = pygame.sprite.Group()
enemy2_group = pygame.sprite.Group()
boss1_group = pygame.sprite.Group()
bullet_refill_group = pygame.sprite.Group()
health_refill_group = pygame.sprite.Group()
double_refill_group = pygame.sprite.Group()
meteor_group = pygame.sprite.Group()
extra_score_group = pygame.sprite.Group()
black_hole_group = pygame.sprite.Group()

boss1_bullets = pygame.sprite.Group()

boss1_health = 150
boss1_health_bar_rect = pygame.Rect(0, 0, 150, 5)
boss1_spawned = False

bg_y_shift = -HEIGHT
background_img = pygame.image.load('images/bg/background.jpg').convert()
background_img2 = pygame.image.load('images/bg/background2.png').convert()
background_img3 = pygame.image.load('images/bg/background3.png').convert()
background_img4 = pygame.image.load('images/bg/background4.png').convert()
background_top = background_img.copy()
current_image = background_img
new_background_activated = False

explosion_images = [pygame.image.load(f"images/explosion/explosion{i}.png") for i in range(8)]

enemy1_img = [
    pygame.image.load('images/enemy/enemy1_1.png').convert_alpha(),
    pygame.image.load('images/enemy/enemy1_2.png').convert_alpha(),
    pygame.image.load('images/enemy/enemy1_3.png').convert_alpha()
]
boss1_img = pygame.image.load('images/boss/boss1.png').convert_alpha()

health_refill_img = pygame.image.load('images/refill/health_refill.png').convert_alpha()

meteor_imgs = [
    pygame.image.load('images/meteors/meteor_1.png').convert_alpha(),
    pygame.image.load('images/meteors/meteor_2.png').convert_alpha(),
    pygame.image.load('images/meteors/meteor_3.png').convert_alpha(),
    pygame.image.load('images/meteors/meteor_4.png').convert_alpha()
]
meteor2_imgs = [
    pygame.image.load('images/meteors/meteor2_1.png').convert_alpha(),
    pygame.image.load('images/meteors/meteor2_2.png').convert_alpha(),
    pygame.image.load('images/meteors/meteor2_3.png').convert_alpha(),
    pygame.image.load('images/meteors/meteor2_4.png').convert_alpha()
]
extra_score_img = pygame.image.load('images/score/score_coin.png').convert_alpha()
black_hole_imgs = [
    pygame.image.load('images/hole/black_hole.png').convert_alpha(),
    pygame.image.load('images/hole/black_hole2.png').convert_alpha()
]

initial_player_pos = (WIDTH // 2, HEIGHT - 100)

score = 0
hi_score = 0
player = Player()
player_life = 200
bullet_counter = 200

paused = False
running = True

joystick = None
if pygame.joystick.get_count() > 0:
    joystick = pygame.joystick.Joystick(0)
    joystick.init()

if show_menu:
    import menu
    menu.main()

is_shooting = False
last_shot_time = 0


while running:

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE and not paused:
                if bullet_counter > 0 and pygame.time.get_ticks() - last_shot_time > SHOOT_DELAY:
                    last_shot_time = pygame.time.get_ticks()
                    bullet = Bullet(player.rect.centerx, player.rect.top)
                    bullets.add(bullet)
                    bullet_counter -= 1
                is_shooting = True

        elif event.type == pygame.KEYUP:
            if event.key == pygame.K_SPACE and player.original_image is not None:
                player.image = player.original_image.copy()
                is_shooting = False
            elif not paused:
                if event.key == pygame.K_LEFT:
                    player.stop_left()
                elif event.key == pygame.K_RIGHT:
                    player.stop_right()
                elif event.key == pygame.K_UP:
                    player.stop_up()
                elif event.key == pygame.K_DOWN:
                    player.stop_down()

    if pygame.time.get_ticks() - last_shot_time > SHOOT_DELAY and is_shooting and not paused:
        if bullet_counter > 0:
            last_shot_time = pygame.time.get_ticks()
            bullet = Bullet(player.rect.centerx, player.rect.top)
            bullets.add(bullet)
            bullet_counter -= 1

    if joystick:
        if not paused:
            move_player_with_joystick(joystick, player)

    if paused:
        font = pygame.font.SysFont('Comic Sans MS', 40)
        text = font.render("PAUSE", True, (255, 255, 255))
        text_rect = text.get_rect(center=(WIDTH / 2, HEIGHT / 2))
        screen.blit(text, text_rect)
        pygame.display.flip()
        continue

    keys = pygame.key.get_pressed()

    if not paused:
        move_player(keys, player)

        screen.blit(current_image, (0, bg_y_shift))
        background_top_rect = background_top.get_rect(topleft=(0, bg_y_shift))
        background_top_rect.top = bg_y_shift + HEIGHT
        screen.blit(background_top, background_top_rect)

    bg_y_shift += 1
    if bg_y_shift >= 0:
        bg_y_shift = -HEIGHT

    if score > 3000:
        bg_y_shift += 2

    if score >= 3000 and not new_background_activated:
        current_image = background_img2
        background_top = background_img2.copy()
        new_background_activated = True

    if score == 0:
        current_image = background_img
        background_top = background_img.copy()
        new_background_activated = False

    screen.blit(current_image, (0, bg_y_shift))
    background_top_rect = background_top.get_rect(topleft=(0, bg_y_shift))
    background_top_rect.top = bg_y_shift + HEIGHT
    screen.blit(background_top, background_top_rect)

    if score > hi_score:
        hi_score = score

    if random.randint(0, 120) == 0:
        enemy_img = random.choice(enemy1_img)
        enemy_object = Enemy1(
            random.randint(100, WIDTH - 50),
            random.randint(-HEIGHT, -50),
            enemy_img,
        )
        enemy1_group.add(enemy_object)

    if score >= 5000 and not boss1_spawned:
        pygame.mixer.Sound('game_sounds/warning.mp3').play()
        boss1_img = boss1_img
        boss1_object = Boss1(
            random.randint(200, WIDTH - 100),
            random.randint(-HEIGHT, -100),
            boss1_img,
        )
        boss1_group.add(boss1_object)
        boss1_spawned = True

    if random.randint(0, 60) == 0:
        extra_score = ExtraScore(
            random.randint(50, WIDTH - 50),
            random.randint(-HEIGHT, -50 - extra_score_img.get_rect().height),
            extra_score_img,
        )

        extra_score_group.add(extra_score)

    if score > 3000 and random.randint(0, 100) == 0:
        meteor_img = random.choice(meteor_imgs)
        meteor_object = Meteors(
            random.randint(0, 50),
            random.randint(0, 50),
            meteor_img,
        )
        meteor_group.add(meteor_object)

    if score > 1000 and random.randint(0, 500) == 0:
        black_hole_img = random.choice(black_hole_imgs)
        black_hole_object = BlackHole(
            random.randint(100, WIDTH - 50),
            random.randint(-HEIGHT, -50 - black_hole_img.get_rect().height),
            black_hole_img,
        )
        black_hole_group.add(black_hole_object)

    if player_life <= 0:
        show_game_over(score)
        boss1_spawned = False
        boss1_health = 150
        boss2_spawned = False
        boss2_health = 150
        boss3_spawned = False
        boss3_health = 200
        score = 0
        player_life = 200
        bullet_counter = 200
        player.rect.topleft = initial_player_pos
        bullets.empty()
        bullet_refill_group.empty()
        health_refill_group.empty()
        double_refill_group.empty()
        extra_score_group.empty()
        black_hole_group.empty()
        meteor_group.empty()
        enemy1_group.empty()
        boss1_group.empty()
        explosions.empty()
        explosions2.empty()

    for black_hole_object in black_hole_group:
        black_hole_object.update()
        black_hole_object.draw(screen)

        if black_hole_object.rect.colliderect(player.rect):
            player_life -= 1
            black_hole_object.sound_effect.play()

    for bullet_refill in bullet_refill_group:

        bullet_refill.update()
        bullet_refill.draw(screen)

        if player.rect.colliderect(bullet_refill.rect):
            if bullet_counter < 200:
                bullet_counter += 50
                if bullet_counter > 200:
                    bullet_counter = 200
                bullet_refill.kill()
                bullet_refill.sound_effect.play()
            else:
                bullet_refill.kill()
                bullet_refill.sound_effect.play()

    for health_refill in health_refill_group:
        health_refill.update()
        health_refill.draw(screen)

        if player.rect.colliderect(health_refill.rect):
            if player_life < 200:
                player_life += 50
                if player_life > 200:
                    player_life = 200
                health_refill.kill()
                health_refill.sound_effect.play()
            else:
                health_refill.kill()
                health_refill.sound_effect.play()


    for meteor_object in meteor_group:
        meteor_object.update()
        meteor_object.draw(screen)

        if meteor_object.rect.colliderect(player.rect):
            player_life -= 10
            explosion = Explosion(meteor_object.rect.center, explosion_images)
            explosions.add(explosion)
            meteor_object.kill()
            score += 50

        bullet_collisions = pygame.sprite.spritecollide(meteor_object, bullets, True)
        for bullet_collision in bullet_collisions:
            explosion = Explosion(meteor_object.rect.center, explosion_images)
            explosions.add(explosion)
            meteor_object.kill()
            score += 80

            if random.randint(0, 10) == 0:
                double_refill = DoubleRefill(
                    meteor_object.rect.centerx,
                    meteor_object.rect.centery,
                    double_refill_img,
                )
                double_refill_group.add(double_refill)

    for enemy_object in enemy1_group:
        enemy_object.update(enemy1_group)
        enemy1_group.draw(screen)

        if enemy_object.rect.colliderect(player.rect):
            player_life -= 10
            explosion = Explosion(enemy_object.rect.center, explosion_images)
            explosions.add(explosion)
            enemy_object.kill()
            score += 20

        bullet_collisions = pygame.sprite.spritecollide(enemy_object, bullets, True)
        for bullet_collision in bullet_collisions:
            explosion = Explosion(enemy_object.rect.center, explosion_images)
            explosions.add(explosion)
            enemy_object.kill()
            score += 50

            if random.randint(0, 8) == 0:
                bullet_refill = BulletRefill(
                    enemy_object.rect.centerx,
                    enemy_object.rect.centery,
                    bullet_refill_img,
                )
                bullet_refill_group.add(bullet_refill)

            if random.randint(0, 8) == 0:
                health_refill = HealthRefill(
                    random.randint(50, WIDTH - 30),
                    random.randint(-HEIGHT, -30),
                    health_refill_img,
                )
                health_refill_group.add(health_refill)

    for boss1_object in boss1_group:
        boss1_object.update(boss1_bullets, player)
        boss1_group.draw(screen)
        boss1_bullets.update()
        boss1_bullets.draw(screen)

        if boss1_object.rect.colliderect(player.rect):
            player_life -= 20
            explosion = Explosion2(boss1_object.rect.center, explosion2_images)
            explosions2.add(explosion)

        bullet_collisions = pygame.sprite.spritecollide(boss1_object, bullets, True)
        for bullet_collision in bullet_collisions:
            explosion2 = Explosion(boss1_object.rect.center, explosion2_images)
            explosions2.add(explosion2)
            boss1_health -= 5
            if boss1_health <= 0:
                explosion = Explosion2(boss1_object.rect.center, explosion3_images)
                explosions.add(explosion)
                boss1_object.kill()
                score += 400

                if random.randint(0, 20) == 0:
                    double_refill = DoubleRefill(
                        boss1_object.rect.centerx,
                        boss1_object.rect.centery,
                        double_refill_img,
                    )
                    double_refill_group.add(double_refill)

        for boss1_bullet in boss1_bullets:
            if boss1_bullet.rect.colliderect(player.rect):
                player_life -= 20
                explosion = Explosion(player.rect.center, explosion3_images)
                explosions.add(explosion)
                boss1_bullet.kill()

        if boss1_health <= 0:
            explosion = Explosion2(boss1_object.rect.center, explosion2_images)
            explosions2.add(explosion)
            boss1_object.kill()

    if boss1_group:
        boss1_object = boss1_group.sprites()[0]
        boss1_health_bar_rect.center = (boss1_object.rect.centerx, boss1_object.rect.top - 5)
        pygame.draw.rect(screen, (255, 0, 0), boss1_health_bar_rect)
        pygame.draw.rect(screen, (0, 255, 0), (boss1_health_bar_rect.left, boss1_health_bar_rect.top, boss1_health, boss1_health_bar_rect.height))

    player_image_copy = player.image.copy()
    screen.blit(player_image_copy, player.rect)

    player_life_surface = pygame.Surface((200, 25), pygame.SRCALPHA, 32)
    player_life_surface.set_alpha(216)

    player_life_bar_width = int(player_life / 200 * 200)
    player_life_bar_width = max(0, min(player_life_bar_width, 200))

    player_life_bar = pygame.Surface((player_life_bar_width, 30), pygame.SRCALPHA, 32)
    player_life_bar.set_alpha(216)

    life_bar_image = pygame.image.load("images/life_bar.png").convert_alpha()

    if player_life > 50:
        player_life_bar.fill((152, 251, 152))
    else:
        player_life_bar.fill((0, 0, 0))

    player_life_surface.blit(life_bar_image, (0, 0))
    player_life_surface.blit(player_life_bar, (35, 0))

    life_x_pos = 10
    screen.blit(player_life_surface, (life_x_pos, 10))

    bullet_counter_surface = pygame.Surface((200, 25), pygame.SRCALPHA, 32)
    bullet_counter_surface.set_alpha(216)
    bullet_counter_bar = pygame.Surface(((bullet_counter / 200) * 200, 30), pygame.SRCALPHA, 32)
    bullet_counter_bar.set_alpha(216)
    bullet_bar_image = pygame.image.load("images/bullet_bar.png").convert_alpha()
    if bullet_counter > 50:
        bullet_counter_bar.fill((255, 23, 23))
    else:
        bullet_counter_bar.fill((0, 0, 0))
    bullet_counter_surface.blit(bullet_bar_image, (0, 0))
    bullet_counter_surface.blit(bullet_counter_bar, (35, 0))
    bullet_x_pos = 10
    bullet_y_pos = player_life_surface.get_height() + 20
    screen.blit(bullet_counter_surface, (bullet_x_pos, bullet_y_pos))

    score_surface = pygame.font.SysFont('Comic Sans MS', 30).render(f'{score}', True, (238, 232, 170))
    score_image_rect = score_surface.get_rect()
    score_image_rect.x, score_image_rect.y = WIDTH - score_image_rect.width - extra_score_img.get_width() - 10, 10

    screen.blit(extra_score_img, (score_image_rect.right + 5, score_image_rect.centery - extra_score_img.get_height()//2))
    screen.blit(score_surface, score_image_rect)

    hi_score_surface = pygame.font.SysFont('Comic Sans MS', 20).render(f'HI-SCORE: {hi_score}', True, (255, 255, 255))
    hi_score_surface.set_alpha(128)
    hi_score_x_pos = (screen.get_width() - hi_score_surface.get_width()) // 2
    hi_score_y_pos = 0
    screen.blit(hi_score_surface, (hi_score_x_pos, hi_score_y_pos))

    pygame.display.flip()

    clock.tick(FPS)

pygame.mixer.music.stop()
pygame.quit()
sys.exit()
```
## **Gameplay Screenshot**

Below are some gameplay screenshots showing different stages and enemy encounters.
![image](https://github.com/user-attachments/assets/b7068efd-792a-42d9-adc9-b12d5537fd77)

![image](https://github.com/user-attachments/assets/2630ad5b-e9b1-4d2f-87d8-2ed2aebae866)





## **Results**

The game successfully uses AI techniques to enhance gameplay, with enemies that change behavior based on the player’s actions. FSMs allow for structured state changes, and rule-based systems provide basic but effective enemy reactions. The pathfinding and dynamic difficulty adjustments make boss battles more challenging, adjusting gameplay complexity as the player progresses.
