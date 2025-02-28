import pygame
import sys
import threading

# Initialize pygame
pygame.init()
clock = pygame.time.Clock()

HEIGHT = 1000
WIDTH = 1000
# Create game window
screen = pygame.display.set_mode([WIDTH,HEIGHT])
pygame.display.set_caption("Red Circle")


class Ball:
    def __init__(self,x,y,radius,color,mass,y_speed,x_speed, id):
        self.x= x
        self.y=y
        self.color=color
        self.radius=radius
        self.color=color
        self.mass=mass
        self.y_speed=y_speed
        self.x_speed = x_speed
        self.id=id
        self.circle=''
        self.vertical_limit=False
        self.horizontal_limit=True

    def draw(self):
        self.circle = pygame.draw.circle(screen, self.color, (self.x, self.y), self.radius)

    def check_gravity(self):
        if self.y<HEIGHT-self.radius-(walls_thickness/2):
            self.y_speed+=gravity
        else:
            self.y_speed=self.y_speed * -1
        return self.y_speed

    def check_direction(self):
        if self.x >= WIDTH - self.radius - (walls_thickness / 2) or self.x <= self.radius + (walls_thickness / 2):
            self.x_speed = self.x_speed * -1
        return self.x_speed


    def update_pos(self):
        self.y+=self.y_speed
        self.x+=self.x_speed

def draw_walls():
    left = pygame.draw.line(screen, 'white', (0,0), (0,HEIGHT), walls_thickness)
    right = pygame.draw.line(screen, 'white', (WIDTH, 0), (WIDTH, HEIGHT), walls_thickness)
    top =  pygame.draw.line(screen, 'white', (0, 0), (WIDTH, 0), walls_thickness)
    bottom = pygame.draw.line(screen, 'white', (0, HEIGHT), (WIDTH, HEIGHT), walls_thickness)
    wall_list=[left,right,bottom,top]
    return wall_list

class Player:
    def __init__(self,x_center_mass, y_center_mass):
        self.x_center_mass=x_center_mass
        self.y_center_mass=y_center_mass
        self.y_arrow=HEIGHT
        self.x_arrow=x_center_mass
        self.bool=False

    def move_left(self):
        self.x_center_mass-=7

    def move_right(self):
        self.x_center_mass += 7

    def draw_character(self):
        pygame.draw.line(screen, 'white', (self.x_center_mass, self.y_center_mass), (self.x_center_mass + angle, HEIGHT), walls_thickness)
        pygame.draw.line(screen, 'white', (self.x_center_mass, self.y_center_mass), (self.x_center_mass - angle, HEIGHT), walls_thickness)
        triangle_points = [(self.x_center_mass+30, self.y_center_mass), (self.x_center_mass-30, self.y_center_mass), (self.x_center_mass, self.y_center_mass-50)]
        pygame.draw.polygon(screen,  'white', triangle_points, 0)
        pygame.draw.circle(screen, 'white', (self.x_center_mass, self.y_center_mass-70),20)

    def shot_arrow(self):
        if player.y_arrow <= 0:
            pygame.draw.line(screen, 'black', (self.x_arrow, HEIGHT-walls_thickness), (self.x_arrow, 0+walls_thickness), arrow_thickness)
            player.y_arrow=HEIGHT
            player.bool=False
            return
        pygame.draw.line(screen, 'white',(self.x_arrow, HEIGHT), (self.x_arrow, self.y_arrow), arrow_thickness)
        self.y_arrow = self.y_arrow-arrow_speed


#game variables
walls_thickness=10
arrow_thickness=4
ball1=Ball(500, 500, 30, 'red', 100, 0, 2,1 )
player=Player(500, 975)
gravity=0.5
arrow_speed=10
angle=30


running = True
while running:
    clock.tick(60)
    screen.fill('black')
    walls = draw_walls()
    ball1.draw()
    ball1.update_pos()
    ball1.y_speed = ball1.check_gravity()
    ball1.x_speed = ball1.check_direction()
    player.draw_character()

    if player.bool:
        player.shot_arrow()
        
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False  # Stops the loop properly

    # Continuous key press check (must be outside the event loop)
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT]:
        player.move_left()
    if keys[pygame.K_RIGHT]:
        player.move_right()
    if keys[pygame.K_SPACE] and not player.bool:
        player.x_arrow = player.x_center_mass
        player.bool = True

    pygame.display.flip()  # Update display

pygame.quit()
sys.exit()  # ✅ Exits the program cleanly
