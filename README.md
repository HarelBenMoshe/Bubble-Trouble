import time
from dbm import error

import pygame
import sys
import threading
import math

# Initialize pygame
pygame.init()
game_over=False
clock = pygame.time.Clock()
HEIGHT = 1000
WIDTH = 1000
# Define font for game-over message
font = pygame.font.Font(None, 50)  # None = default font, size 50

# Create game window
screen = pygame.display.set_mode([WIDTH, HEIGHT])
pygame.display.set_caption("Red Circle")


def draw_walls():
    left = pygame.draw.line(screen, 'green', (0, 0), (0, HEIGHT), walls_thickness)
    right = pygame.draw.line(screen, 'green', (WIDTH, 0), (WIDTH, HEIGHT), walls_thickness)
    top = pygame.draw.line(screen,'green', (0, 0), (WIDTH, 0), walls_thickness)
    bottom = pygame.draw.line(screen, 'green', (0, HEIGHT), (WIDTH, HEIGHT), walls_thickness)
    wall_list = [left, right, bottom, top]
    return wall_list


class Ball:
    def __init__(self, x, y, radius, color, mass, y_speed, x_speed):
        self.x = x
        self.y = y
        self.color = color
        self.radius = radius
        self.color = color
        self.mass = mass
        self.y_speed = y_speed
        self.x_speed = x_speed
        self.id = id
        self.circle = ''
        self.vertical_limit = False
        self.horizontal_limit = True

    def draw(self):
        if self.color != 'black':
            self.circle = pygame.draw.circle(screen, self.color, (self.x, self.y), self.radius)

    def check_gravity(self):
        if self.y < HEIGHT - self.radius - (walls_thickness / 2):
            self.y_speed += gravity
        else:
            self.y_speed = self.y_speed * -1
        return self.y_speed

    def check_direction(self):
        if self.x >= WIDTH - self.radius - (walls_thickness / 2) or self.x <= self.radius + (walls_thickness / 2):
            self.x_speed = self.x_speed * -1
        return self.x_speed

    def update_pos(self):
        self.y += self.y_speed
        self.x += self.x_speed

    def check_collision(self):
        global game_over

        for i in range(0, 360, 5):  # Check full circle
            radians = math.radians(i)
            x_pixel = int(min(max(self.x + self.radius * math.cos(radians), 0), WIDTH - 1))
            y_pixel = int(min(max(self.y + self.radius * math.sin(radians), 0), HEIGHT - 1))
            color = screen.get_at((x_pixel, y_pixel))[:3]  # Get RGB color
            if color == (255, 165, 0):  # Orange color of character
                game_over = True
        if abs(player.x_arrow - self.x) <= self.radius and self.radius+self.y >= player.y_arrow:
            return True  # Arrow hit ball
        return False

    def ball_explosion(self):
        if self.radius > 10:  # Only split if the ball is big enough
            new_radius = self.radius // 2  # Each new ball is half the size
            new_speed = abs(self.y_speed)  # Ensure the new balls move upwards

            # Create two new balls with opposite x directions
            ball_left, ball_right = (
                Ball(self.x - new_radius, self.y, new_radius, 'blue', 50, -new_speed, -2),
                Ball(self.x + new_radius, self.y, new_radius, 'blue', 50, -new_speed, 2,)
            )
            return [ball_left, ball_right]  # Return the new balls
        return []  # If the ball is too small, return an empty list (disappear)


class Player:
    def __init__(self, x_center_mass, y_center_mass, character_color):
        self.x_center_mass = x_center_mass
        self.y_center_mass = y_center_mass
        self.y_arrow = HEIGHT
        self.arrow_color = 'white'
        self.character_color = character_color
        self.x_arrow = x_center_mass
        self.collision = False
        self.fire=False

    def move_left(self):
        self.x_center_mass = max(self.x_center_mass - 7, 30)  # Ensures it doesn't go left of 30

    def move_right(self):
        self.x_center_mass = min(self.x_center_mass + 7, WIDTH - 30)  # Ensures it doesn't go right of WIDTH-30

    def draw_character(self):
        pygame.draw.line(screen, self.character_color, (self.x_center_mass, self.y_center_mass), (self.x_center_mass + angle, HEIGHT), walls_thickness)
        pygame.draw.line(screen, self.character_color, (self.x_center_mass, self.y_center_mass),(self.x_center_mass - angle, HEIGHT), walls_thickness)
        triangle_points = [(self.x_center_mass + 25, self.y_center_mass), (self.x_center_mass - 25, self.y_center_mass),(self.x_center_mass, self.y_center_mass - 50)]
        pygame.draw.polygon(screen, self.character_color, triangle_points, 0)
        pygame.draw.rect(screen, self.character_color,(self.x_center_mass - 10, self.y_center_mass - 60, 20, 20))

    def handle_arrow(self):
        if self.y_arrow<=0:
            self.arrow_color = 'black'
        else:
            for ball in balls[:]:
                if ball.check_collision():
                    new_balls = ball.ball_explosion()  # Get new smaller balls
                    balls.remove(ball)  # Remove original ball
                    balls.extend(new_balls)  # Add new balls to the list
                    player.fire = False
                    self.arrow_color = 'black'
                    break
        pygame.draw.line(screen, self.arrow_color, (self.x_arrow, HEIGHT), (self.x_arrow, self.y_arrow),arrow_thickness)
        if self.arrow_color=='black':
            self.arrow_color = 'white'
            self.y_arrow=HEIGHT
            player.fire=False
        else:
            self.y_arrow = self.y_arrow - arrow_speed


# game variables
walls_thickness = 10
arrow_thickness = 4
ball = Ball(500, 500, 30, 'red', 100, 0, 2)
player = Player(500, 975, 'orange')
gravity = 0.5
arrow_speed = 15
angle = 30


balls = [Ball(500, 500, 100, 'red', 100, 0, 2)]  # Start with one big ball

running = True
while running:
    clock.tick(60)
    screen.fill('black')
    walls = draw_walls()
    player.draw_character()
    # Iterate over a copy of the list to prevent modification errors during iteration
    for ball in balls[:]:
        ball.draw()
        ball.update_pos()
        ball.y_speed = ball.check_gravity()
        ball.x_speed = ball.check_direction()
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT]:
        player.move_left()
    if keys[pygame.K_RIGHT]:
        player.move_right()
    if keys[pygame.K_SPACE] and not player.fire:
        player.x_arrow = player.x_center_mass
        player.fire = True
    if player.fire:
        player.handle_arrow()
    pygame.display.flip()  # Update display

    for ball in balls[:]:
        ball.check_collision()  # checks if the ball hit the character
        if game_over:
            pygame.draw.line(screen, 'black', (player.x_arrow, HEIGHT-walls_thickness), (player.x_arrow, player.y_arrow),arrow_thickness)
            player.draw_character()
            text_surface = font.render("Ouch! You lost!", True, (255, 255, 255))
            screen.blit(text_surface, (WIDTH // 2 - 100, HEIGHT // 2))
            pygame.display.flip()  

            # Allow quitting during game over screen
            wait_start = pygame.time.get_ticks()
            waiting = True
            while waiting and pygame.time.get_ticks() - wait_start < 3000:  # 3 seconds wait
                for event in pygame.event.get():
                    if event.type == pygame.QUIT or (event.type == pygame.KEYDOWN and event.key == pygame.K_ESCAPE):
                        waiting = False
                        running = False
                pygame.time.delay(100)  # Small delay to prevent CPU hogging
            running = False
            continue
            
pygame.quit()
sys.exit()
