import pygame
from pygame import mixer
from pygame.locals import *
import random
import cv2
import mediapipe as mp
import numpy as np
import sys
sys.path.append('E:/Coding/Python code')

pygame.mixer.pre_init(44100,-16,2,512)
mixer.init()
pygame.init()

clock = pygame.time.Clock()
fps = 60

screen_width = 1215
screen_height = 700

screen = pygame.display.set_mode((screen_width, screen_height))
camera_surface = pygame.Surface((480, 270))
pygame.display.set_caption('Space Invaders')

font30 = pygame.font.SysFont('Constantia',30)
font40 = pygame.font.SysFont('Constantia',40)
font50 = pygame.font.SysFont('Ariel Black',50)

explosion_fx = pygame.mixer.Sound("explosion.wav")
explosion_fx.set_volume(0.25)

explosion2_fx = pygame.mixer.Sound("explosion2.wav")
explosion2_fx.set_volume(0.25)

laser_fx = pygame.mixer.Sound("laser.wav")
laser_fx.set_volume(0.25)

music_on = True
sound_on = True

rows = 3
cols = 11
alien_cooldown = 2000
last_alien_shot = pygame.time.get_ticks()
countdown = 3
last_count = pygame.time.get_ticks()
game_over = 0

red = (255, 0, 0)
green = (0, 255, 0)
white = (255, 255, 255)

bg = pygame.image.load("bg.png")

pygame.mixer.music.load("background_music.mp3")
pygame.mixer.music.set_volume(0.5)
pygame.mixer.music.play(-1)

mp_hands = mp.solutions.hands
hands = mp_hands.Hands(min_detection_confidence=0.7, min_tracking_confidence=0.7)

cap = cv2.VideoCapture(0)

def draw_bg():
    screen.blit(bg, (0, 0))

def draw_text(text, font, text_col, x, y):
    img = font.render(text, True, text_col)
    screen.blit(img, (x, y))

class Spaceship(pygame.sprite.Sprite):
    def __init__(self, x, y, health):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.image.load("spaceship.png")
        self.rect = self.image.get_rect()
        self.rect.center = [x, y]
        self.health_start = health
        self.health_remaining = health
        self.last_shot = pygame.time.get_ticks()

    def update(self):
        speed = 8
        cooldown = 1000
        game_over = 0

        key = pygame.key.get_pressed()
        if key[pygame.K_LEFT] and self.rect.left > 0:
            self.rect.x -= speed
        if key[pygame.K_RIGHT] and self.rect.right < screen_width:
            self.rect.x += speed
        if key[pygame.K_UP] and self.rect.top < screen_height:
            self.rect.y -= speed
        if key[pygame.K_DOWN] and self.rect.bottom > 0:
            self.rect.y += speed

        ret, frame = cap.read()

        if ret:
            gesture = detect_gesture(frame)
            if gesture == "LEFT" :
                    spaceship.rect.x -= speed
            elif gesture == "RIGHT" :
                    spaceship.rect.x += speed
            elif gesture == "SPACE":
                time_now = pygame.time.get_ticks()
                if time_now - spaceship.last_shot > 1000:
                        laser_fx.play()
                        bullet = Bullets(spaceship.rect.centerx, spaceship.rect.top)
                        bullet_group.add(bullet)
                        spaceship.last_shot = time_now

        time_now =pygame.time.get_ticks()

        if key[pygame.K_SPACE] and time_now - self.last_shot > cooldown:
            laser_fx.play()
            bullet = Bullets(self.rect.centerx, self.rect.top)
            bullet_group.add(bullet)
            self.last_shot = time_now

        self.mask = pygame.mask.from_surface(self.image)

        pygame.draw.rect(screen, red, (self.rect.x, (self.rect.bottom + 10), self.rect.width, 15))
        if self.health_remaining > 0:
            pygame.draw.rect(screen, green, (self.rect.x, (self.rect.bottom + 10), int(self.rect.width * (self.health_remaining/ self.health_start)),15))
        elif self.health_remaining <=0:
            explosion = Explosion(self.rect.centerx, self.rect.centery, 3)
            explosion_group.add(explosion)
            self.kill()
            game_over = -1
        return game_over

class Bullets(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.image.load("bullet.png")
        self.rect = self.image.get_rect()
        self.rect.center = [x, y]

    def update(self):
        self.rect.y -= 5
        if self.rect.bottom < 0:
            self.kill()
        if pygame.sprite.spritecollide(self, alien_group, True):
            self.kill()
            explosion_fx.play()
            explosion = Explosion(self.rect.centerx, self.rect.centery, 2)
            explosion_group.add(explosion)

class Aliens(pygame.sprite.Sprite):
	def __init__(self, x, y):
		pygame.sprite.Sprite.__init__(self)
		self.image = pygame.image.load("alien" + str(random.randint(1, 5)) + ".png")
		self.rect = self.image.get_rect()
		self.rect.center = [x, y]
		self.move_counter = 0
		self.move_direction = 1

	def update(self):
		self.rect.x += self.move_direction
		self.move_counter += 1
		if abs(self.move_counter) > 75:
			self.move_direction *= -1
			self.move_counter *= self.move_direction

class Alien_Bullets(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.image.load("alien_bullet.png")
        self.rect = self.image.get_rect()
        self.rect.center = [x, y]

    def update(self):
        self.rect.y +=2
        if self.rect.top > screen_height:
            self.kill()
        if pygame.sprite.spritecollide(self, spaceship_group, False, pygame.sprite.collide_mask):
            self.kill()
            explosion2_fx.play()

            spaceship.health_remaining -= 1
            explosion = Explosion(self.rect.centerx, self.rect.centery, 1)
            explosion_group.add(explosion)

class Explosion(pygame.sprite.Sprite):
    def __init__(self, x, y, size):
        pygame.sprite.Sprite.__init__(self)
        self.images = []
        for num in range(1, 6):
            img = pygame.image.load(f"exp{num}.png")
            if size == 1:
                img = pygame.transform.scale(img, (20, 20))
            if size == 2:
                img = pygame.transform.scale(img, (40, 40))
            if size == 3:
                img = pygame.transform.scale(img, (160, 160))

            self.images.append(img)

        self.index = 0
        self.image = self.images[self.index]
        self.rect = self.image.get_rect()
        self.rect.center = [x, y]
        self.counter = 0

    def update(self):
        explosion_speed = 3

        self.counter += 1

        if self.counter >= explosion_speed and self.index < len(self.images) - 1:
            self.counter = 0
            self.index += 1
            self.image = self.images[self.index]

        if self.index >= len(self.images) - 1 and self.counter >= explosion_speed:
            self.kill()


spaceship_group = pygame.sprite.Group()
bullet_group = pygame.sprite.Group()
alien_group = pygame.sprite.Group()
alien_bullet_group = pygame.sprite.Group()
explosion_group = pygame.sprite.Group()

def menu():
    draw_bg
    draw_text("SPACE INVADERS", font40, green, screen_width // 2 - 150, 100)
    buttons ={
        "Play": (screen_width // 2-50, 300),
        "Guide": (screen_width// 2-50, 600),
        "Music": (screen_width// 2-50, 400),
        "Sound": (screen_width// 2-50, 500)
    }
    for buttons_text, (x, y) in buttons.items(): 
        pygame.draw.rect(screen, white, (x - 10, y - 10, 150, 50), 2)
        if buttons_text in ["Music", "Sound"]:
            pygame.draw.rect(screen, (0, 0, 0), (x + 80, y, 50, 30))  
            status = "ON" if (music_on if buttons_text == "Music" else sound_on) else "OFF"
            draw_text(f"{buttons_text}", font30, white, x, y)
        else:
            draw_text(buttons_text, font30, white, x, y)




buttons = {
    "Play": (screen_width // 2 - 50, 300),
    "Music": (screen_width // 2 - 50, 400),
    "Sound": (screen_width // 2 - 50, 500),
    "Guide": (screen_width // 2 - 50, 600),
}

def draw_back_to_menu_button():
    pygame.draw.rect(screen, white, (10, 10, 100, 40), 2)
    draw_text("Menu", font30, white, 20, 15)

def create_aliens():
    for row in range(rows):
        for item in range(cols):
            alien = Aliens(100 + item * 100, 100 + row*70)
            alien_group.add(alien)

def reset_game():
    global spaceship, spaceship_group, bullet_group, alien_group, alien_bullet_group, explosion_group, game_over, countdown, last_alien_shot
    spaceship_group.empty()
    bullet_group.empty()
    alien_group.empty()
    alien_bullet_group.empty()
    explosion_group.empty()
    game_over = 0
    countdown = 3
    last_alien_shot = pygame.time.get_ticks()
    
    # Tạo lại tàu vũ trụ
    spaceship = Spaceship(int(screen_width / 2), screen_height - 100, 3)
    spaceship_group.add(spaceship)
    
    # Tạo lại kẻ địch
    create_aliens()

def detect_gesture(frame):
    frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = hands.process(frame_rgb)
    gesture = None

    if results.multi_hand_landmarks and len(results.multi_hand_landmarks) <= 2:
        hand_status = {"Left": None, "Right": None}

        for hand_landmarks, hand_label in zip(
            results.multi_hand_landmarks, results.multi_handedness
        ):
            handedness = hand_label.classification[0].label  # "Left" or "Right"
            landmarks = [(lm.x, lm.y) for lm in hand_landmarks.landmark]
            wrist = landmarks[mp_hands.HandLandmark.WRIST]
            index_tip = landmarks[mp_hands.HandLandmark.INDEX_FINGER_TIP]
            thumb_tip = landmarks[mp_hands.HandLandmark.THUMB_TIP]
            pinky_tip = landmarks[mp_hands.HandLandmark.PINKY_TIP]

            # Determine if the hand is open (distance between thumb and pinky is large)
            distance_thumb_pinky = np.linalg.norm(np.array(thumb_tip) - np.array(pinky_tip))
            hand_status[handedness] = "OPEN" if distance_thumb_pinky > 0.2 else "CLOSED"

        # Gesture recognition based on hand statuses
        left_hand = hand_status.get("Left")
        right_hand = hand_status.get("Right")


        if left_hand == "OPEN" :
            gesture = "RIGHT"
        elif right_hand == "OPEN" :
            gesture = "LEFT"
        elif left_hand == "CLOSED" and right_hand == "CLOSED":
            gesture = "SPACE"

    return gesture

create_aliens()

spaceship = Spaceship(int(screen_width / 2), screen_height -100, 3)
spaceship_group.add(spaceship)



menu_active = True  # Menu là trạng thái mặc định
run = True


while run:
    clock.tick(fps)

    if menu_active:
        draw_bg
        menu()
        # Event handlers for menu
        for event in pygame.event.get():
            if event.type == pygame.KEYDOWN and event.key == pygame.K_ESCAPE:
                menu_active = False
                run = False
            if event.type == pygame.MOUSEBUTTONDOWN:
                mouse_x, mouse_y = pygame.mouse.get_pos()
                for btn_text, (x, y) in buttons.items():
                    if x - 10 <= mouse_x <= x + 110 and y - 10 <= mouse_y <= y + 40:
                        if btn_text == "Play":
                            menu_active = False
                            if game_over !=0:
                                reset_game()
                        elif btn_text == "Sound":
                            sound_on = not sound_on
                            explosion_fx.set_volume(0.25 if sound_on else 0)
                            explosion2_fx.set_volume(0.25 if sound_on else 0)
                            laser_fx.set_volume(0.25 if sound_on else 0)
                        elif btn_text == "Music":
                            music_on = not music_on
                            if music_on:
                                pygame.mixer.music.play(-1)
                            else:
                                pygame.mixer.music.stop()
                        elif btn_text == "Guide":
                            # Show guide
                            draw_bg()
                            draw_text("Guide:", font40, white, screen_width // 2 - 50, 100)
                            draw_text("Press arrow keys to move, SPACE to shoot, ESC to pause game.", font50, red, 100, 200)
                            draw_text("Dodge alien bullets and shot them", font50, red, 100, 250)
                            draw_text("When all enemy killed or your helth become 0, game will end", font50, red, 100, 300)
                            draw_back_to_menu_button()
                            pygame.display.update()
                            guide_active = True
                            while guide_active:
                                for guide_event in pygame.event.get():
                                    if guide_event.type == pygame.MOUSEBUTTONDOWN:
                                        mouse_x, mouse_y = pygame.mouse.get_pos()
                                    if 10 <= mouse_x <= 160 and 10 <= mouse_y <= 50:
                                        guide_active = False
                                        draw_bg()

        pass

    else:
        # Gameplay logic
        draw_bg()
        for event in pygame.event.get():
            if event.type == pygame.KEYDOWN and event.key == pygame.K_ESCAPE:
                menu_active = True
                draw_bg

        if countdown == 0:
            # Create random alien bullets

            time_now = pygame.time.get_ticks()
            if time_now - last_alien_shot > alien_cooldown and len(alien_bullet_group) < 5 and len(alien_group) > 0:
                attacking_alien = random.choice(alien_group.sprites())
                alien_bullet = Alien_Bullets(attacking_alien.rect.centerx, attacking_alien.rect.bottom)
                alien_bullet_group.add(alien_bullet)
                last_alien_shot = time_now

            if len(alien_group) == 0:
                game_over = 1

            if game_over == 0:
                game_over = spaceship.update()

                bullet_group.update()
                alien_group.update()
                alien_bullet_group.update()
            else:
                if game_over == -1:
                    draw_text('GAME OVER!', font40, white, int(screen_width / 2 - 100), int(screen_height / 2 + 50))
                if game_over == 1:
                    draw_text('YOU WIN!', font40, white, int(screen_width / 2 - 100), int(screen_height / 2 + 50))
                draw_back_to_menu_button()
                if event.type == pygame.MOUSEBUTTONDOWN:
                    mouse_x, mouse_y = pygame.mouse.get_pos()
                    if 10 <= mouse_x <= 160 and 10 <= mouse_y <= 50:
                        menu_active = True
                        draw_bg()

        if countdown > 0:
            draw_text('GET READY!', font40, white, int(screen_width / 2 - 110), int(screen_height / 2 + 50))
            draw_text(str(countdown), font40, white, int(screen_width / 2 - 10), int(screen_height / 2 + 100))
            count_timer = pygame.time.get_ticks()

            if count_timer - last_count > 1000:
                countdown -= 1
                last_count = count_timer

        explosion_group.update()

        spaceship_group.draw(screen)
        bullet_group.draw(screen)
        alien_group.draw(screen)
        alien_bullet_group.draw(screen)
        explosion_group.draw(screen)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False

    pygame.display.update()

pygame.quit()
