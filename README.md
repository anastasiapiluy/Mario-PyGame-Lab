# Mario-PyGame-Lab

### Архив с фотографиями

[Использованные спрайты]()


### Программа

```python
import random

import pygame

pygame.init()
screenWidth, screenHeight = 900, 600
fullWidth=1200
screen = pygame.display.set_mode((screenWidth, screenHeight))
clock = pygame.time.Clock()

marioImgL = pygame.image.load("marioLeft.png").convert_alpha()
marioImgL = pygame.transform.scale(marioImgL, (50, 50))
marioImgR = pygame.image.load("marioRight.png").convert_alpha()
marioImgR = pygame.transform.scale(marioImgR, (50,50))
monster1ImgL = pygame.image.load("monster1Left.png").convert_alpha()
monster1ImgL = pygame.transform.scale(monster1ImgL, (40,40))
monster1ImgR = pygame.image.load("monster1Right.png").convert_alpha()
monster1ImgR = pygame.transform.scale(monster1ImgR, (40,40))
monster2ImgL = pygame.image.load("monster2Left.png").convert_alpha()
monster2ImgL = pygame.transform.scale(monster2ImgL, (40,40))
monster2ImgR = pygame.image.load("monster2Right.png").convert_alpha()
monster2ImgR = pygame.transform.scale(monster2ImgR, (40,40))
heartImg = pygame.image.load("heart.png").convert_alpha()
heartImg = pygame.transform.scale(heartImg, (20,20))
coinImg = pygame.image.load("coin.png").convert_alpha()
coinImg = pygame.transform.scale(coinImg, (25,25))
doorImg = pygame.image.load("door.png").convert_alpha()
doorImg = pygame.transform.scale(doorImg, (80,80))

speed = 10
size = 50
gravity = 0.65
jump = -15
lives = 3
moving = 0
direction = 1

timer = 0
state = "play"
done = False

ground = 80
firstLevel = 130
secLevel = 280
thLevel = 430

door = None
doorSize = 80

coinSize = 25
coinHeight = 20
coinCounter = 0
cns = 0

monsterSize = 40
monsterSpeed = 2

font = pygame.font.SysFont("Arial", 20)
over = pygame.font.SysFont("Arial", 80)

x = 50
y = screenHeight-(size+ground)
velY = 0
standing = False

def createLevel(y):
    indent = 100
    if random.random()<0.2:
        return [(indent,y,fullWidth-2*indent,20)]
    gap1, gap2 = random.randint(100,150), random.randint(100,150)

    total = fullWidth-gap1-gap2-2*indent

    left = random.randint(140,total-2*140)
    center = random.randint(140, total-left-140)
    right=total-left-center

    return[(indent,y,left,20),(indent+left+gap1,y,center,20), (indent+left+gap1+center+gap2,y,right,20)]

def generateLevel():
    global platforms,direction, coins,cns, monster1, monster2,door,x,y,velY, standing, coinCounter, lives, state, timer
    platforms = [(0,screenHeight-ground,fullWidth,ground)]
    levels = [screenHeight-ground-firstLevel,screenHeight-ground-secLevel,screenHeight-ground-thLevel]
    for i in levels:
        platforms += createLevel(i)

    top = min(platforms[1:], key = lambda p: p[1])
    door = pygame.Rect(top[0]+top[2]//2-doorSize//2, top[1]-doorSize,doorSize,doorSize)

    x = 50
    y = screenHeight-(size+ground)
    velY = 0
    standing = False

    coins = generateCoins(platforms)
    cns = len(coins)
    coinCounter = 0
    lives = 3
    direction = 1
    monster1, monster2 = generateMonsters(platforms)
    state = "play"
    timer = 0

def generateCoins(platforms):
    amount = min(random.randint(3,5), len(platforms))
    coins = []
    pl = random.sample(platforms, amount)
    for (x,y,w,h) in pl:
        coinX = random.randint(x + coinSize//2, x + w - coinSize//2)
        coinY = y-coinHeight
        coins.append((coinX, coinY))
    return coins

def collect():
    global coins, coinCounter
    mario = pygame.Rect(x,y,size,size)
    ls = []
    for c in coins:
        if not mario.collidepoint(c[0],c[1]):
            ls.append(c)
    coinCounter += len(coins) - len(ls)
    coins = ls

def generateMonsters(platforms):
    platf = [p for p in platforms if p[3]!= ground]
    platf = random.sample(platf, 2)
    x,y,w,h = platf[0]
    first = [x+w//2-monsterSize//2,y-monsterSize,1,x,x+w-monsterSize,1]
    x,y,w,h = platf[1]
    second = [x+w//2-monsterSize//2,y-monsterSize,0, False,1]
    return first,second

def updateF(m):
    m[0]+=monsterSpeed*m[2]
    if m[0]<=m[3]:
        m[0] = m[3]
        m[2]=1
        m[5] = 1
    elif m[0]>=m[4]:
        m[0]=m[4]
        m[2]=-1
        m[5] = -1

def updateS(m,platforms):
    if m[0]<x:
        m[0]+=monsterSpeed
        m[4] = 1
    elif m[0]>x:
        m[0]-=monsterSpeed
        m[4] = -1
    if m[3] and y<m[1]-5 and abs(m[0]-x)<200:
        m[2] = jump
        m[3] = False
    m[2] += gravity
    m[1] += m[2]
    m[3] = False
    for (monX,monY,monW,monH) in platforms:
        if m[0]+monsterSize > monX and m[0] < monX+monW and m[1]+monsterSize > monY and m[1] < monY+monH:
            if m[2] >= 0 and m[1]+monsterSize-m[2] <= monY:
                m[1] = monY-monsterSize
                m[2] = 0
                m[3] = True

def monsterCollision():
    global lives,state,timer
    if timer > 0:
        return
    mario = pygame.Rect(x,y,size,size)
    mon1 = pygame.Rect(monster1[0], monster1[1], monsterSize, monsterSize)
    mon2 = pygame.Rect(monster2[0], monster2[1], monsterSize, monsterSize)
    if mario.colliderect(mon1) or mario.colliderect(mon2):
        lives-=1
        if lives<=0:
            state = "lose"
        else:
            timer = 90

def win():
    global state
    if len(coins)>0:
        return
    mario = pygame.Rect(x,y,size,size)
    if door and mario.colliderect(door):
        state = "win"

def restart():
    global x,y,velY,moving, standing,direction,cns, lives, coins,coinCounter, monster1, monster2, state,timer
    x = 50
    y = screenHeight-(size+ground)
    velY=0
    moving=0
    standing = False
    lives = 3
    direction=1
    timer = 0
    coinCounter = 0
    state = "play"
    coins = generateCoins(platforms)
    cns = len(coins)
    monster1,monster2 = generateMonsters(platforms)

generateLevel()

while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                if state == "win":
                    generateLevel()
            elif event.key == pygame.K_r:
                restart()
            elif event.key == pygame.K_UP and standing and state=="play":
                keys = pygame.key.get_pressed()
                if not keys[pygame.K_LEFT] and not keys[pygame.K_RIGHT]:
                    velY = jump
                    standing = False

    if state == "play":
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and standing:
            x-=speed
            direction = -1
        elif keys[pygame.K_RIGHT] and standing:
            x+=speed
            direction = 1

        velY += gravity
        y += velY
        standing = False
        for (px,py,pw,ph) in platforms:
            if x+size > px and x<px+pw and y+size>py and y<py+ph:
                if velY >= 0 and y+size-velY <= py:
                    y = py - size
                    velY = 0
                    standing = True

        x = max(0,min(x,fullWidth-size))
        moving = x-150
        if moving<0:
            moving = 0
        elif moving>(fullWidth-screenWidth):
            moving = fullWidth-screenWidth
        collect()
        updateF(monster1)
        updateS(monster2,platforms)

        if timer > 0:
            timer-=1

        monsterCollision()
        win()

    screen.fill((170, 170, 255))
    for p in platforms:
        pygame.draw.rect(screen, (100,255,100) if p[3]==ground else (100,50,50), pygame.Rect(p[0]-moving, p[1], p[2], p[3]))
    for (coinX,coinY) in coins:
        screen.blit(coinImg,(coinX-moving-coinSize//2, coinY-coinSize//2))
    if door:
        imgDoor = doorImg
        screen.blit(imgDoor, (door[0]-moving, door[1]))
    imgMonster1 = monster1ImgR if monster1[5] == 1 else monster1ImgL
    screen.blit(imgMonster1, (monster1[0]-moving, monster1[1]))
    imgMonster2 = monster2ImgR if monster2[4] == 1 else monster2ImgL
    screen.blit(imgMonster2, (monster2[0]-moving, monster2[1]))
    img = marioImgR if direction==1 else marioImgL
    if timer%10<5:
        screen.blit(img, (x-moving, y))
    for i in range(lives):
        screen.blit(heartImg, (630+i*30,14))
    text = font.render(f"Монет: {coinCounter}/{cns}", True, (0,0,0))
    screen.blit(text, (500,10))
    if state=="lose":
        bigText = over.render("You lose!", True, (255,0,0))
        screen.blit(bigText, (450-bigText.get_width()//2,400))
        smallText = font.render("Press R to restart", True, (255, 255, 255))
        screen.blit(smallText, (450 - smallText.get_width() // 2, 485))
    elif state=="win":
        bigText = over.render("You win!", True, (255, 0, 0))
        screen.blit(bigText, (450 - bigText.get_width() // 2, 400))
        smallText = font.render("Press SPACE to generate different level", True, (255,255,255))
        screen.blit(smallText, (450-smallText.get_width() // 2, 485))

    pygame.display.update()
    clock.tick(60)

pygame.quit()
```
