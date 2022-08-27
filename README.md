import pygame
from pygame.locals import QUIT, KEYDOWN, KEYUP, K_RIGHT, K_LEFT, K_SPACE, K_b
import sys
from random import randint as ri

pygame.init()

size = 400, 800
SUR = pygame.display.set_mode(size)
pygame.display.set_caption("Shooting!")
FPS = pygame.time.Clock()


class Obj:
    def __init__(self):
        self.x, self.y = 0, 0
        self.sp = ri(1,6)
    def putpic(self, pic):
        self.img = pygame.image.load(pic)
    def setsize(self, x, y):
        self.img = pygame.transform.scale(self.img, (x,y))
        self.sx, self.sy = x, y
    def show(self):
        SUR.blit(self.img, (self.x, self.y))

ship = Obj()
ship.putpic("rocket.png")
ship.setsize(55,110)
ship.x, ship.y = size[0]//2 - ship.sx//2 , size[1]-ship.sy-15

beams = []
rocks = []

bnum = 10
score = 0
inter = 0
game_over = False
go_right, go_left, shoot = (False, ) * 3
mess = pygame.font.SysFont(None, 50)
bmess = pygame.font.SysFont(None, 40)

def crash(a,b):
    if a.x - b.sx < b.x < a.x + a.sx:
        if a.y - b.sy < b.y < a.y + a.sy:
            return True
    return False


while True:

    SUR.fill((0,0,0))
    for i in pygame.event.get():
        if i.type == QUIT:
            pygame.quit()
            sys.exit()
        elif i.type == KEYDOWN:
            if i.key == K_RIGHT:
                go_right = True
            elif i.key == K_LEFT:
                go_left = True
            elif i.key == K_SPACE:
                inter = 0
                shoot = True
            elif i.key == K_b:
                bnum = 10
        elif i.type == KEYUP:
            if i.key == K_RIGHT:
                go_right = False
            elif i.key == K_LEFT:
                go_left = False
            elif i.key == K_SPACE:
                shoot = False

    if not game_over:
        if go_right:
            ship.x += 5
            if ship.x > size[0]-ship.sx:
                ship.x = size[0]-ship.sx
        
        if go_left:
            ship.x -= 5
            if ship.x < 0:
                ship.x = 0

        if shoot and inter % 10 == 0 and bnum:
            beam = Obj()
            beam.putpic("star.png")
            beam.setsize(25,40)
            beam.x = ship.x + ship.sx//2 - beam.sx//2
            beam.y = ship.y - beam.sy + 20
            beams.append(beam)
            bnum -= 1

        if ri(1,100) == 1:
            rock = Obj()
            rock.putpic("rock.png")
            if ri(1,2) == 1:
                rock.setsize(60,30)
            else:
                rock.setsize(100,60)
            rock.setsize(50,30)
            rock.x = ri(0, size[0]-rock.sx)
            rock.y = -rock.sy
            rocks.append(rock)

        inter += 1
        
        for i in beams:
            i.y -= 15
            i.show()
            if i.y < -i.sy:
                del beams[beams.index(i)]
            

        for i in rocks:
            i.y += i.sp
            i.show()
            if i.y > size[1]:
                del rocks[rocks.index(i)]


        for i in beams:
            for j in rocks:
                if crash(i,j):
                    score += 100 + j.sp * 10
                    print(score)
                    del beams[beams.index(i)]
                    del rocks[rocks.index(j)]
        for i in rocks:
            if crash(i, ship):
                game_over = True


        if bnum > 5:
            b = bmess.render(f"BEAM {bnum}", True, (255,255,255)) 
        else:
            b = bmess.render(f"BEAM {bnum}", True, (255,0,0)) 
        bx = b.get_width()
        SUR.blit(b, (size[0]-bx-30, 30))
        ship.show()


    else:
        m = mess.render(f"YOUR SCORE {score}!!", True, (255,0,0))
        mx, my = m.get_width(), m.get_height()
        SUR.blit(m, (size[0]//2-mx//2, size[1]//2-my//2))

    pygame.display.flip()
    FPS.tick(60)
