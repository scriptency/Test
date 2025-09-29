import pygame
import sys
import math

# Initialize Pygame
pygame.init()

# Constants
TILE_SIZE = 40
MAP_WIDTH = 20
MAP_HEIGHT = 20
WINDOW_WIDTH = TILE_SIZE * MAP_WIDTH
WINDOW_HEIGHT = TILE_SIZE * MAP_HEIGHT
FPS = 60
VISION_RADIUS = 1 # tiles

# Colors
WHITE  = (255, 255, 255)
GRAY  = (100, 100, 100)  # wall
BLACK  = (0, 0, 0)
GREEN  = (0, 255, 0)    # player
BROWN  = (139, 69, 19)   # gold door
ORANGE = (205, 133, 63)   # gold key
LIGHTGRAY = (200, 200, 200) # silver key
DARKGRAY = (70, 70, 70)   # silver door
RED   = (255, 0, 0)    # text

# Tile codes:
# 0 = wall, 1 = floor
# 2 = gold door, 3 = gold key
# 4 = silver door, 5 = silver key
dungeon_map = [
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,1,1,1,0,1,1,1,1,1,1,1,1,1,1,1,1,3,1,0], # gold key
  [0,1,1,1,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,1,1,0,0,0,0,0,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,1,0,0,1,1,1,0,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,1,1,0,1,1,1,0,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,1,1,1,1,1,1,0,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,0],
  [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0],
  [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0],
  [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,1,1,1,1,1,1,2,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,1,1,1,0,0,0,0,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,1,1,1,0,5,1,0,1,1,1,1,1,1,1,0],
  [0,1,1,1,0,1,1,1,2,1,1,0,1,1,1,1,0,0,0,0], # gold door
  [0,1,1,1,0,0,0,0,0,0,0,0,1,1,1,1,0,1,1,0], # silver key
  [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,1,1,0],
  [0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,4,1,1,0], # silver door
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
]

# Player
player_pos = [1, 1]
keys = set() # {"gold", "silver"}
message = ""
message_timer = 0

# Font
font = pygame.font.SysFont("Arial", 24)

# Screen
screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Dungeon with Multiple Keys & Doors")

# Clock
clock = pygame.time.Clock()

def draw_map():
  for y in range(MAP_HEIGHT):
    for x in range(MAP_WIDTH):
      rect = pygame.Rect(x*TILE_SIZE, y*TILE_SIZE, TILE_SIZE, TILE_SIZE)
      tile = dungeon_map[y][x]
      if tile == 0:
        pygame.draw.rect(screen, GRAY, rect)
      elif tile == 1:
        pygame.draw.rect(screen, WHITE, rect)
      elif tile == 2: # gold door
        pygame.draw.rect(screen, BROWN, rect)
      elif tile == 3: # gold key
        pygame.draw.rect(screen, ORANGE, rect)
      elif tile == 4: # silver door
        pygame.draw.rect(screen, DARKGRAY, rect)
      elif tile == 5: # silver key
        pygame.draw.rect(screen, LIGHTGRAY, rect)

def draw_player():
  rect = pygame.Rect(player_pos[1]*TILE_SIZE, player_pos[0]*TILE_SIZE, TILE_SIZE, TILE_SIZE)
  pygame.draw.rect(screen, GREEN, rect)

def move_player(dy, dx):
  global message, message_timer
  new_y = player_pos[0] + dy
  new_x = player_pos[1] + dx
  if 0 <= new_y < MAP_HEIGHT and 0 <= new_x < MAP_WIDTH:
    tile = dungeon_map[new_y][new_x]

    if tile == 1: # floor
      player_pos[:] = [new_y, new_x]

    elif tile == 2: # gold door
      if "gold" in keys:
        dungeon_map[new_y][new_x] = 1
        player_pos[:] = [new_y, new_x]
        message, message_timer = "Gold Door Unlocked!", 120

    elif tile == 3: # gold key
      keys.add("gold")
      dungeon_map[new_y][new_x] = 1
      player_pos[:] = [new_y, new_x]
      message, message_timer = "Gold Key Collected!", 120

    elif tile == 4: # silver door
      if "silver" in keys:
        dungeon_map[new_y][new_x] = 1
        player_pos[:] = [new_y, new_x]
        message, message_timer = "Silver Door Unlocked!", 120

    elif tile == 5: # silver key
      keys.add("silver")
      dungeon_map[new_y][new_x] = 1
      player_pos[:] = [new_y, new_x]
      message, message_timer = "Silver Key Collected!", 120

def draw_fog():
    fog = pygame.Surface((WINDOW_WIDTH, WINDOW_HEIGHT), pygame.SRCALPHA)
    fog.fill((0, 0, 0, 220))  #whitefhds

    
    px = player_pos[1] * TILE_SIZE + TILE_SIZE // 2
    py = player_pos[0] * TILE_SIZE + TILE_SIZE // 2

    
    mx, my = pygame.mouse.get_pos()

    
    angle_to_mouse = math.atan2(my - py, mx - px)

    
    cone_angle = math.radians(60)     
    vision_length = TILE_SIZE * 8     
    step = 2                         

    
    points = [(px, py)]
    start_angle = angle_to_mouse - cone_angle / 2
    end_angle = angle_to_mouse + cone_angle / 2

    for angle in [start_angle + math.radians(a) for a in range(0, int(math.degrees(cone_angle)), step)]:
        dx = math.cos(angle)
        dy = math.sin(angle)
        ray_x, ray_y = px, py

        for i in range(vision_length):
            tile_x = int(ray_x // TILE_SIZE)
            tile_y = int(ray_y // TILE_SIZE)

            
            if 0 <= tile_x < MAP_WIDTH and 0 <= tile_y < MAP_HEIGHT:
                if dungeon_map[tile_y][tile_x] == 0:  
                    break
            else:
                break

            ray_x += dx
            ray_y += dy

        points.append((ray_x, ray_y))

   
    if len(points) > 2:
        pygame.draw.polygon(fog, (0, 0, 0, 0), points)

    
    small_radius = TILE_SIZE * 1  
    pygame.draw.circle(fog, (0, 0, 0, 0), (px, py), int(small_radius))

    
    screen.blit(fog, (0, 0))

def draw_hud():
  global message_timer
  key_text = font.render(f"Keys: {', '.join(keys) if keys else 'None'}", True, RED)
  screen.blit(key_text, (10, 5))

  if message_timer > 0:
    msg_text = font.render(message, True, RED)
    screen.blit(msg_text, (200, 5))
    message_timer -= 1

# Game Loop
running = True
while running:
    clock.tick(FPS)
    screen.fill(BLACK)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_w: move_player(-1, 0)
            elif event.key == pygame.K_s: move_player(1, 0)
            elif event.key == pygame.K_a: move_player(0, -1)
            elif event.key == pygame.K_d: move_player(0, 1)

    draw_map()
    draw_player()
    draw_fog()
    draw_hud()

    pygame.display.flip()

pygame.quit()
sys.exit()

