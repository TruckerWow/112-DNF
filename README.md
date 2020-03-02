# 112-DNF
## Description: 
This game is similar to the arcade version of the combat game. The player needs to protect the base under the attack of the enemy wave after wave, once the base is destroyed by the enemy, the game fails.

Run tp3.py in the code folder to start the game.

The goal is to protect the base here from being destroyed by enemies, and enemies will occur randomly from both sides. They will automatically trace and attack the player. The player will fall to the ground and his health point also decreases when getting attacked. And this red ball represents the player's health point.

The player has four methods to attack the enemy. The first one is normal attack by pressing 'x', which costs nothing. And the skill bar contains three skills. we need to consume magic point to use skills and the blue ball represents the player's magic point. We could press 'a' to use first skill and attack enemies around the player; press 's' to launch a magic ball in a straight line; and press 'd' to attack all enemies present. When we kill the enemies, some coins will occur there, and we need to get close to collect the coins.

If the player is killed by enemies, it needs 5 seconds to revive the player. During this period, enemies will continue to attack the base, and the base will automatically attack enemies within a certain range. Also, if the base is destroyed, the game is over. The red bars above enemies and the base represent their health point.

We could also press 'b' to open the store, and use coins to buy medicine in this store. After buying that, we could use red medicine to restore health point by pressing '1' or use blue one to restore magic point by pressing '2'.

Press 'm' to open property panel, here we could see all properties of the player, the score and the attack value could be increased by killing enemies.

At the left-top corner is a game timer, when the time is up, we could get into next level.

In level 2, we could control two characters. The enemy is always flying in the sky, and randomly attack players at long range. So the players has to predict the position of the enemy and use parabola to throw the explosive to attack the enemy and adjust the throw angle by pressing 'up''down' or 'w' 's'. 

Also, the players could use skill by pressing 'f' to freeze the enemy. 

When we kill this enemy, we win.

***************************************************************************************************************************************

```
#cmu_112_graphics from 
#https://www.cs.cmu.edu/~112/notes/notes-animations-part1.html
from cmu_112_graphics import *
from tkinter import *
from PIL import Image
import random
import math


class KnightsTemplar(ModalApp):
    def appStarted(app):
        app.splashScreenMode = SplashScreenMode()
        app.gameMode = GameMode()
        app.gameMode2 = GameMode2()
        app.winMode = WinMode()
        app.loseMode = LoseMode()
        app.setActiveMode(app.splashScreenMode)

class SplashScreenMode(Mode):
    def appStarted(mode):
        mode.playerChooseIndex = 0
        mode.spriteCounterChoose = 0
        mode.playerChoose = playerChoose(mode)
        mode.player1Index = 1
        mode.player2Index = 2

    def mousePressed(mode, event):
        x, y = event.x, event.y
        #click player to choose player
        if mode.playerChoose.player1X - mode.playerChoose.player1Width//2\
            < x < mode.playerChoose.player1X + \
            mode.playerChoose.player1Width//2 and \
            mode.playerChoose.player1Y - mode.playerChoose.player1Height//2\
            < y < mode.playerChoose.player1Y + \
            mode.playerChoose.player1Height//2:
            mode.playerChooseIndex = mode.player1Index
        elif mode.playerChoose.player2X - mode.playerChoose.player2Width//2\
            < x < mode.playerChoose.player2X + \
            mode.playerChoose.player2Width//2 and \
            mode.playerChoose.player2Y - mode.playerChoose.player2Height//2\
            < y < mode.playerChoose.player2Y + \
            mode.playerChoose.player2Height//2:
            mode.playerChooseIndex = mode.player2Index
        #click start button to start game after choosing player
        elif (mode.playerChooseIndex == mode.player1Index or\
            mode.playerChooseIndex == mode.player2Index) and\
            mode.playerChoose.startX - mode.playerChoose.startWidth//2\
            < x < mode.playerChoose.startX + \
            mode.playerChoose.startWidth//2 and \
            mode.playerChoose.startY - mode.playerChoose.startHeight//2\
            < y < mode.playerChoose.startY + \
            mode.playerChoose.startHeight//2:
            mode.app.setActiveMode(mode.app.gameMode)
        
    def pictureCount(mode):
        mode.spriteCounterChoose = (1 + mode.spriteCounterChoose)\
                            % len(mode.playerChoose.chooseSprites)

    def timerFired(mode):
        mode.pictureCount()

    def redrawAll(mode, canvas):
        mode.playerChoose.draw(canvas)

class WinMode(Mode):
    def appStarted(mode):
        #image from https://dnf.qq.com/
        win = mode.loadImage('winAndLose.jpg')
        mode.win = mode.scaleImage(win,1.78)

    def redrawAll(mode, canvas):
        canvas.create_image(mode.width//2,mode.height//2,\
                           image=ImageTk.PhotoImage(mode.win))
        canvas.create_text(mode.width//2, mode.height//2, fill='white', text=\
                          'Knights of Honor', font='Arial 27 bold')
        canvas.create_text(mode.width//2, mode.height//2+50, fill='white',\
                           text='You Win', font='Arial 27 bold')


class LoseMode(Mode):
    def appStarted(mode):
        #image from https://dnf.qq.com/
        win = mode.loadImage('winAndLose.jpg')
        mode.win = mode.scaleImage(win,1.78)

    def redrawAll(mode, canvas):
        canvas.create_image(mode.width//2,mode.height//2,\
                           image=ImageTk.PhotoImage(mode.win))
        canvas.create_text(mode.width//2, mode.height//2, fill='white', text=\
                          'Knight\'s Evening', font='Arial 27 bold')
        canvas.create_text(mode.width//2, mode.height//2+50, fill='white',\
                           text='You Lose', font='Arial 27 bold')


class GameMode(Mode):
    def appStarted(mode):
        mode.timerDelay = 100
        mode.spriteCounter = 0
        mode.spriteCounterAttack = 0
        mode.spriteCounterGetAttack = 0
        mode.spriteCounterBase = 0
        mode.spriteCounterBaseAttack = 0
        mode.spriteCounterEnemy1 = 0
        mode.spriteCounterEnemy1Attack = 0
        mode.spriteCounterEnemy1GetAttack = 0
        mode.spriteCounterEnemy2 = 0
        mode.spriteCounterEnemy2Attack = 0
        mode.spriteCounterEnemy2GetAttack = 0
        mode.spriteCounterSkill1 = 0
        mode.spriteCounterSkill2 = 0
        mode.spriteCounterSkill3 = 0
        mode.spriteCounterPlayerSkill3 = 0
        mode.limitY1 = 0.58 * mode.height
        mode.limitY2 = 0.90 * mode.height
        mode.skillCounter = 0
        mode.player1 = player1(mode)
        mode.player2 = player2(mode)
        mode.gameMap = gameMap(mode)
        mode.base = base(mode)
        mode.enemy1 = enemy1(mode)
        mode.enemy2 = enemy2(mode)
        mode.skill = skill(mode)
        mode.skill1 = skill1(mode)
        mode.skill2 = skill2(mode)
        mode.skill3 = skill3(mode)
        mode.hpMedicine = hpMedicine(mode)
        mode.mpMedicine = mpMedicine(mode)
        mode.store = store(mode)
        mode.propertyPanel = propertyPanel(mode)
        #mode.coin = coin()
        mode.deadCount = 0
        mode.enemySet = set()
        mode.enemyRemoveSet = set()
        mode.enemySetY = dict()
        mode.coins = set()
        mode.timer = 0
        mode.totalTime = 500
        mode.levels = 1
        mode.score = 0
        for i in range(2):
            mode.enemySet.add(enemy1(mode))
            mode.enemySet.add(enemy2(mode))
            #mode.enemySetY[enemy1(mode)] = mode.enemy1.y
        
    def timerFired(mode):
        #print(mode.player1.status)
        mode.timer += 1
        TimeToNextLevel = (mode.totalTime-mode.timer)//(1000//mode.timerDelay)
        if mode.timer % (1000//mode.timerDelay*7) == 0:
            for i in range(2):
                mode.enemySet.add(enemy1(mode))
                mode.enemySet.add(enemy2(mode))
        if TimeToNextLevel == 0: #go to next level
            mode.app.setActiveMode(mode.app.gameMode2)
        if mode.player1.status == mode.player1.skill1 or \
            mode.player2.status == mode.player2.skill1:
            mode.skillCounter += 2
        mode.revivePlayer()
        mode.pictureCount()
        mode.enemyMove()
        mode.resetPlayerStatus()
        mode.skill2.skillMove()
        if mode.player1.status == mode.player1.skill1 or \
            mode.player2.status == mode.player2.skill1:
            mode.skill1.stopSkill1()
        elif mode.player1.status == mode.player1.skill3 or \
            mode.player2.status == mode.player2.skill3:
            mode.skill3.stopSkill3()
        mode.player1.die()
        mode.player2.die()
        mode.skill2.resetPosition()
        #mode.player1.getInvincible()
        #mode.player2.getInvincible()

    def resetPlayerStatus(mode):
        if mode.app.splashScreenMode.playerChooseIndex ==\
            mode.app.splashScreenMode.player1Index:
            mode.player1.getCoin() #get close to coin to grap coin
            mode.player1.statusTime += 1
            if mode.player1.status == mode.player1.run and \
                mode.player1.statusTime >= 5: #the time player run without press
                mode.player1.status = mode.player1.stand
            elif mode.player1.status == mode.player1.attack and \
                mode.player1.statusTime >= 19:#the time player attack without 'x'
                mode.player1.status = mode.player1.stand
            elif mode.player1.status == mode.player1.skill2 and \
                mode.player1.statusTime >= 3:#the time player attack without 'x'
                mode.player1.status = mode.player1.stand
        elif mode.app.splashScreenMode.playerChooseIndex ==\
            mode.app.splashScreenMode.player2Index:
            mode.player2.getCoin() #get close to coin to grap coin
            mode.player2.statusTime += 1
            if mode.player2.status == mode.player2.run and \
                mode.player2.statusTime >= 5: #the time player run without press
                mode.player2.status = mode.player2.stand
            elif mode.player2.status == mode.player2.attack and \
                mode.player2.statusTime >= 19:#the time player attack without 'x'
                mode.player2.status = mode.player2.stand
            elif mode.player2.status == mode.player2.skill2 and \
                mode.player2.statusTime >= 3:#the time player attack without 'x'
                mode.player2.status = mode.player2.stand
    
    def revivePlayer(mode):
        if mode.player1.status == mode.player1.dead or\
            mode.player2.status == mode.player1.dead:
            mode.deadCount += 1
            if mode.deadCount >= 50: #player to revive
                mode.player1.hp = mode.player1.fullHp
                mode.player2.hp = mode.player2.fullHp
                mode.player1.x = mode.width//5
                mode.player1.y = mode.height//2+mode.player1.height//2
                mode.player2.x = mode.width//5
                mode.player2.y = mode.height//2+mode.player1.height//2
                mode.player1.status = mode.player1.stand
                mode.player2.status = mode.player2.stand
                mode.deadCount = 0

    def enemyMove(mode):
        mode.enemyRemoveSet = set()
        for enemy in mode.enemySet:
            #print(enemy.hp)
            enemy.getStatus()
            enemy.fallToGround()
            #enemy run to player or base 
            if enemy.status == enemy.run or enemy.status == enemy.runToBase:
                enemy.move()
            if enemy.status == enemy.attack:
                if mode.app.splashScreenMode.playerChooseIndex ==\
                    mode.app.splashScreenMode.player1Index:
                    mode.player1.status = mode.player1.getAttack
                    if type(enemy) == enemy1:#decrease HP
                        mode.player1.hp -= 10
                    if type(enemy) == enemy2:#decrease HP
                        mode.player1.hp -= 20
                elif mode.app.splashScreenMode.playerChooseIndex ==\
                    mode.app.splashScreenMode.player2Index:
                    mode.player2.status = mode.player2.getAttack
                    if type(enemy) == enemy1:#decrease HP
                        mode.player2.hp -= 10
                    if type(enemy) == enemy2:#decrease HP
                        mode.player2.hp -= 20
            if enemy.hp <= 0: #assert enemy dead
                mode.enemyRemoveSet.add(enemy)
                #add coin to coins when enemy get killed
                mode.coins.add(coin(mode,enemy.x,enemy.y+enemy.height//2,\
                    enemy.coinValue))
                mode.score += enemy.scoreValue
                if mode.app.splashScreenMode.playerChooseIndex ==\
                    mode.app.splashScreenMode.player1Index:
                    mode.player1.attackValue += 5
                elif mode.app.splashScreenMode.playerChooseIndex ==\
                    mode.app.splashScreenMode.player2Index:
                    mode.player2.attackValue += 5
                
        #remove dead enemy
        mode.enemySet = mode.enemySet - mode.enemyRemoveSet

    def pictureCount(mode):
        if mode.app.splashScreenMode.playerChooseIndex ==\
            mode.app.splashScreenMode.player1Index:
            mode.spriteCounter = (1 + mode.spriteCounter)\
                                % len(mode.player1.sprites)
            mode.spriteCounterAttack = (1 + mode.spriteCounterAttack)\
                                % len(mode.player1.attackSprites)
            mode.spriteCounterGetAttack = (1 + mode.spriteCounterGetAttack)\
                                % len(mode.player1.getAttackSprites)
        elif mode.app.splashScreenMode.playerChooseIndex ==\
            mode.app.splashScreenMode.player2Index:
            mode.spriteCounter = (1 + mode.spriteCounter)\
                                % len(mode.player2.sprites)
            mode.spriteCounterAttack = (1 + mode.spriteCounterAttack)\
                                % len(mode.player2.attackSprites)
            mode.spriteCounterGetAttack = (1 + mode.spriteCounterGetAttack)\
                                % len(mode.player2.getAttackSprites)
        mode.spriteCounterBase = (1 + mode.spriteCounterBase)\
                            % len(mode.base.baseSprites)
        mode.spriteCounterBaseAttack = (1 + mode.spriteCounterBaseAttack)\
                            % len(mode.base.baseAttackSprites)
        mode.spriteCounterEnemy1 = (1 + mode.spriteCounterEnemy1)\
                            % len(mode.enemy1.enemy1Sprites)
        mode.spriteCounterEnemy1Attack = (1 + mode.spriteCounterEnemy1Attack)\
                            % len(mode.enemy1.enemy1AttackSprites)
        mode.spriteCounterEnemy1GetAttack = \
                            (1 + mode.spriteCounterEnemy1GetAttack)\
                            % len(mode.enemy1.enemy1GetAttackSprites)
        mode.spriteCounterEnemy2 = (1 + mode.spriteCounterEnemy2)\
                            % len(mode.enemy2.enemy2Sprites)
        mode.spriteCounterEnemy2Attack = (1 + mode.spriteCounterEnemy2Attack)\
                            % len(mode.enemy2.enemy2AttackSprites) 
        mode.spriteCounterEnemy2GetAttack = \
                            (1 + mode.spriteCounterEnemy2GetAttack)\
                            % len(mode.enemy2.enemy2GetAttackSprites)
        if mode.skillCounter % 2 == 0:
            if mode.player1.status == mode.player1.skill1 or \
                mode.player2.status == mode.player2.skill1:
                mode.spriteCounterSkill1 = (1 + mode.spriteCounterSkill1)\
                            % len(mode.skill1.skill1Sprites)
            elif mode.player1.status == mode.player1.skill3 or \
                mode.player2.status == mode.player2.skill3: 
                mode.spriteCounterSkill3 = (1 + mode.spriteCounterSkill3)\
                            % len(mode.skill3.skill3Sprites)
                mode.spriteCounterPlayerSkill3 = (1 + \
                            mode.spriteCounterPlayerSkill3)\
                            % len(mode.skill3.skill3PlayerSprites)                  
            mode.spriteCounterSkill2 = (1 + mode.spriteCounterSkill2)\
                            % len(mode.skill2.skill2Sprites)
            

    def keyMove(mode,key,player):
        if key in ['Right','Left','Up','Down']:
            player.statusTime = 0
            player.status = player.run
            if key == 'Right':
                player.right = True
            elif key == 'Left':
                player.right = False
            player.move(key)
        elif key == 'x':
            player.statusTime = 0
            player.status = player.attack
        elif key == 'a' and player.mp >= 10:
            player.mp -= 10
            player.statusTime = 0
            player.status = player.skill1
        elif key == 's' and player.mp >= 20:
            mode.skill2.getDirection()
            mode.skill2.status = True
            player.statusTime = 0
            player.mp -= 20
            player.status = player.skill2
        elif key == 'd' and player.mp >= 10:
            player.mp -= 30
            player.statusTime = 0
            player.status = player.skill3
        elif key == '1':
            player.useHpMedicine()
        elif key == '2':
            player.useMpMedicine()


    def keyPressed(mode, event):
        if mode.player1.status != mode.player1.dead and\
            mode.player2.status != mode.player1.dead:
            if mode.app.splashScreenMode.playerChooseIndex ==\
                mode.app.splashScreenMode.player1Index:
                mode.keyMove(event.key,mode.player1)
            elif mode.app.splashScreenMode.playerChooseIndex ==\
                mode.app.splashScreenMode.player2Index:
                mode.keyMove(event.key,mode.player2)
        if event.key == 'b':
            #open or close store
            mode.store.openStatus = not mode.store.openStatus
        if event.key == 'm':
            #open or close store
            mode.propertyPanel.openStatus\
                 = not mode.propertyPanel.openStatus
    
    def buyMedicine(mode,x,y):
        if mode.app.splashScreenMode.playerChooseIndex ==\
            mode.app.splashScreenMode.player1Index:
                if (mode.store.hpLeft <= x <= mode.store.hpRight) and \
                    (mode.store.hpTop <= y <= mode.store.hpDown):
                    mode.player1.buyHpMedicine()
                elif (mode.store.mpLeft <= x <= mode.store.mpRight) and \
                    (mode.store.mpTop <= y <= mode.store.mpDown):
                    mode.player1.buyMpMedicine()
        elif mode.app.splashScreenMode.playerChooseIndex ==\
            mode.app.splashScreenMode.player2Index:
                if (mode.store.hpLeft <= x <= mode.store.hpRight) and \
                    (mode.store.hpTop <= y <= mode.store.hpDown):
                    mode.player2.buyHpMedicine()
                elif (mode.store.mpLeft <= x <= mode.store.mpRight) and \
                    (mode.store.mpTop <= y <= mode.store.mpDown):
                    mode.player2.buyMpMedicine()
        
    def mousePressed(mode,event):
        x, y = event.x, event.y
        if mode.store.openStatus == True:
            #buy medicine
            mode.buyMedicine(x,y)

    def drawReviveTime(mode,canvas):
        if mode.player1.status == mode.player1.dead or\
            mode.player2.status == mode.player1.dead:
            remindTime = (50-mode.deadCount)//10
            canvas.create_text(mode.width//2,150, fill = 'darkorange',\
                text=f'{remindTime}s to revive',font=f'Arial 28 bold')
    
    def drawTimer(mode,canvas):
        TimeToNextLevel = (mode.totalTime-mode.timer)//(1000//mode.timerDelay)
        canvas.create_text(150,50, fill = 'black',\
                text=f'{TimeToNextLevel}s to next level',font=f'Arial 15 bold')

    def redrawAll(mode, canvas):
        mode.gameMap.draw(canvas)
        #draw coins after enemy get killed
        for coin in mode.coins:
            coin.draw(canvas)
        if mode.app.splashScreenMode.playerChooseIndex ==\
            mode.app.splashScreenMode.player1Index:
            mode.player1.draw(canvas)
        elif mode.app.splashScreenMode.playerChooseIndex ==\
            mode.app.splashScreenMode.player2Index:
            mode.player2.draw(canvas)
        for enemy in mode.enemySet:
            enemy.draw(canvas)
        mode.skill1.draw(canvas)
        mode.base.draw(canvas)
        mode.skill.draw(canvas)
        mode.skill2.draw(canvas)
        mode.skill3.draw(canvas)
        mode.hpMedicine.draw(canvas)
        mode.mpMedicine.draw(canvas)
        mode.store.draw(canvas)
        mode.propertyPanel.draw(canvas)
        mode.drawReviveTime(canvas)
        mode.drawTimer(canvas)

class GameMode2(Mode):
    def appStarted(mode):
        mode.timerDelay = 100
        mode.spriteCounter = 0
        mode.spriteCounterAttack = 0
        mode.spriteCounterGetAttack = 0
        mode.spriteCounterEnemy3 = 0
        mode.spriteCounterEnemy3Attack = 0
        mode.spriteCounterEnemy3GetAttack = 0
        mode.spriteCounterExplosive1 = 0
        mode.spriteCounterExplosive2 = 0
        mode.spriteCounterBase = 0
        mode.spriteCounterBaseAttack = 0
        mode.limitY1 = 0.65 * mode.height
        mode.limitY2 = mode.height
        mode.player1 = player1(mode)
        mode.player2 = player2(mode)
        mode.player2.y = \
            mode.player2.y = mode.height//2+mode.player1.height//2+100
        mode.gameMap = gameMap2(mode)
        mode.base2 = base2(mode)
        mode.enemy3 = enemy3(mode)
        mode.explosive = explosive(mode)
        mode.explosive2 = explosive2(mode)
        mode.hpMedicine = hpMedicine(mode)
        mode.skill = skill(mode)
        mode.skillInLevel2 = skillInLevel2(mode)
        mode.deadCount = 0
        mode.enemySet = set()
        mode.enemyRemoveSet = set()
        mode.timer = 0
        mode.totalTime = 1000
        mode.levels = 1
        mode.score = 0
        mode.runAndAttackCount = 0

    def resetPlayerStatus(mode):
        mode.player1.statusTime += 1
        mode.player2.statusTime += 1
        if mode.player1.status == mode.player1.run and \
            mode.player1.statusTime >= 5: #the time player run without press
            mode.player1.status = mode.player1.stand
        if mode.player2.status == mode.player2.run and \
            mode.player2.statusTime >= 5: #the time player run without press
            mode.player2.status = mode.player2.stand

    def revivePlayer(mode):
        if mode.player1.status == mode.player1.dead or\
            mode.player2.status == mode.player1.dead:
            mode.deadCount += 1
            if mode.deadCount >= 50: #player to revive
                mode.player1.hp = mode.player1.fullHp
                mode.player2.hp = mode.player2.fullHp
                mode.player1.x = mode.width//5
                mode.player1.y = mode.height//2+mode.player1.height//2
                mode.player2.x = mode.width//5
                mode.player2.y = mode.height//2+mode.player1.height//2+100
                mode.player1.status = mode.player1.stand
                mode.player2.status = mode.player2.stand
                mode.deadCount = 0
            
    
    def timerFired(mode):
        #print(mode.player1.status)
        mode.timer += 1
        mode.runAndAttackCount += 1
        mode.resetPlayerStatus()
        mode.pictureCount()
        mode.explosive.getPosition()
        mode.explosive.stopBoom()
        mode.explosive2.stopBoom()
        mode.explosive2.getPosition()
        if mode.enemy3.status != mode.enemy3.frozen:
            mode.enemy3.getStatus()
            mode.enemy3.attackPlayerOnce()
        mode.enemy3.fly()
        mode.enemy3.getRidOfFrozen()
        mode.player1.die()
        mode.player2.die()
        mode.revivePlayer()
        mode.enemy3.getAttackInfrozen()
        

    def player1KeyMove(mode,key,player):
        if key in ['Right','Left','Up','Down']:
            if player.status != player.throw:
                player.status = player.run
            player.statusTime = 0
            if key == 'Up'\
                and player.status == player.throw:
                mode.explosive.angle += math.pi/50
            elif key == 'Down'\
                and player.status == player.throw:
                mode.explosive.angle -= math.pi/50
            elif key == 'Right':
                player.right = True
                if player.status == player.throw:
                    player.status = player.run
            elif key == 'Left':
                player.right = False
                if player.status == player.throw:
                    player.status = player.run
            player.move(key)
        elif key == '1' and\
            player.status != player.throw:
            player.statusTime = 0
            player.status = player.throw
        elif key == '1' and\
            player.status == player.throw:
            mode.explosive.launch = True

    def player2KeyMove(mode,key,player):
        if key in ['d','a','w','s']:
            if player.status != player.throw:
                player.status = player.run
            player.statusTime = 0
            if key == 'w'\
                and player.status == player.throw:
                mode.explosive2.angle += math.pi/50
            elif key == 's'\
                and player.status == player.throw:
                mode.explosive2.angle -= math.pi/50
            elif key == 'd':
                player.right = True
                if player.status == player.throw:
                    player.status = player.run
            elif key == 'a':
                player.right = False
                if player.status == player.throw:
                    player.status = player.run
            player.moveInLevel2(key)
        elif key == 'j' and\
            player.status != player.throw:
            player.statusTime = 0
            player.status = player.throw
        elif key == 'j' and\
            player.status == player.throw:
            mode.explosive2.launch = True


    def keyPressed(mode, event):
        if mode.player1.status != mode.player1.dead and\
            mode.player2.status != mode.player1.dead:
            mode.player1KeyMove(event.key,mode.player1)
            mode.player2KeyMove(event.key,mode.player2)
        if event.key == 'f' and mode.player1.mp >= 25:
            mode.player1.mp -= 25
            mode.player2.mp -= 25
            mode.enemy3.status = mode.enemy3.frozen

    def drawReviveTime(mode,canvas):
        if mode.player1.status == mode.player1.dead or\
            mode.player2.status == mode.player1.dead:
            remindTime = (50-mode.deadCount)//10
            canvas.create_text(mode.width//2,150, fill = 'darkorange',\
                text=f'{remindTime}s to revive',font=f'Arial 28 bold') 
    
    def pictureCount(mode):
        mode.spriteCounterEnemy3Attack = (1 + mode.spriteCounterEnemy3Attack)\
                            % len(mode.enemy3.enemy3AttackSprites)
        mode.spriteCounterEnemy3GetAttack = (1 +\
                mode.spriteCounterEnemy3GetAttack) %\
                len(mode.enemy3.enemy3GetAttackSprites)
        mode.spriteCounterEnemy3 = (1 +\
                mode.spriteCounterEnemy3) % len(mode.enemy3.enemy3Sprites)
        mode.spriteCounter = (1 + mode.spriteCounter)\
                            % len(mode.player1.sprites)
        mode.spriteCounterAttack = (1 + mode.spriteCounterAttack)\
                                % len(mode.player1.attackSprites)
        mode.spriteCounterGetAttack = (1 + mode.spriteCounterGetAttack)\
                                % len(mode.player1.getAttackSprites)
        mode.spriteCounterBase = (1 + mode.spriteCounterBase)\
                            % len(mode.base2.baseSprites)
        mode.spriteCounterBaseAttack = (1 + mode.spriteCounterBaseAttack)\
                            % len(mode.base2.baseAttackSprites)
        if mode.explosive.boom == True:
            mode.spriteCounterExplosive1  = (1 + mode.spriteCounterExplosive1)\
                            % len(mode.explosive.explosiveSprites)
        if mode.explosive2.boom == True:   
            mode.spriteCounterExplosive2  = (1 + mode.spriteCounterExplosive2)\
                            % len(mode.explosive2.explosiveSprites)

    def drawReviveTime(mode,canvas):
        if mode.player1.status == mode.player1.dead or\
            mode.player2.status == mode.player1.dead:
            remindTime = (50-mode.deadCount)//10
            canvas.create_text(mode.width//2,150, fill = 'darkorange',\
                text=f'{remindTime}s to revive',font=f'Arial 28 bold')
    
    def redrawAll(mode, canvas):
        mode.gameMap.draw(canvas)
        mode.player1.draw(canvas)
        mode.player2.draw(canvas)
        mode.skillInLevel2.draw(canvas)
        mode.base2.draw(canvas)
        mode.enemy3.draw(canvas)
        mode.explosive.draw(canvas)
        mode.explosive2.draw(canvas)
        mode.drawReviveTime(canvas)

class player1(object):
    def __init__(self,mode):
        self.mode = mode
        self.width = 130
        self.height = 100
        self.x = mode.width//5
        self.y = mode.height//2+self.height//2
        self.dx = 20
        self.dy = 20
        self.status = 0
        self.right = True
        self.stand = 0
        self.run = 1
        self.attack = 2
        self.getAttack = 3
        self.throw = 20
        self.skillStatus = 0
        self.skill1 = 11
        self.skill2 = 12
        self.skill3 = 13
        self.statusTime = 0
        self.standAndRun()
        self.attackMove()
        self.getAttackMove()
        self.hpImage()
        self.mpImage()
        self.dieMove()
        self.throwMove()
        self.i=0
        self.attackDistance = 150
        self.hp = 1000
        self.fullHp = 1000
        self.mp = 100
        self.fullMp = 100
        self.dead = 4
        self.attackValue = 100
        self.coins = 500
        self.getCoinDistance = 50
        self.invincible = False
        

    def standAndRun(self):
        urlStand = '90.png' #image from http://www.aigei.com/
        self.player1Stand = self.mode.loadImage(urlStand)
        self.player1StandTrans = self.mode.loadImage(urlStand).\
                                transpose(Image.FLIP_LEFT_RIGHT)
        self.sprites = []
        self.spritesTanspose = []
        for i in range(105,113):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'{i}.png') 
            self.sprites.append(image)
            self.spritesTanspose.append(image.transpose(Image.FLIP_LEFT_RIGHT))
        
    def attackMove(self):
        self.attackSprites = []
        self.attackSizes = dict()
        for i in range(188,210):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'{i}.png') 
            self.attackSprites.append(image)
            (width, height) = image.size
            self.attackSizes[i] = (width, height)# get size of every attack

    def move(self,direction):
        if direction == 'Left': 
            self.x -= self.dx
            if self.x < 0:
                self.x = 0
        if direction == 'Right': 
            self.x += self.dx
            if self.x > self.mode.width:
                self.x = self.mode.width
        if direction == 'Up' and self.status != self.throw:
            self.y -= self.dy
            if self.y + self.height//2 < self.mode.limitY1:
                self.y = self.mode.limitY1 - self.height//2
        if direction == 'Down' and self.status != self.throw:
            self.y += self.dy
            if self.y + self.height//2 > self.mode.limitY2:
                self.y = self.mode.limitY2 - self.height//2


    def getAttackMove(self):
        self.getAttackSprites = []
        self.getAttackSizes = dict()
        for i in range(1,11):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'getattack{i}.png') 
            self.getAttackSprites.append(image)
            (width, height) = image.size
            # get size of every getAttack
            self.getAttackSizes[i] = (width, height)

    def getCoin(self):
        coinRemoveSet = set()
        for coin in self.mode.coins:
            if ((self.x - coin.x) ** 2 + \
                ((self.y + self.height//2) - coin.y)**2) ** 0.5\
                <= self.getCoinDistance:
                self.coins += coin.coinValue
                coinRemoveSet.add(coin)
                #print(self.coins)
        self.mode.coins = self.mode.coins - coinRemoveSet
    
    def die(self):
        if self.hp <= 0:
            self.status = self.dead
    
    def dieMove(self):
        #image from http://www.aigei.com/
        self.deadImage = self.mode.loadImage('getattack9.png')
        self.deadImageTrans = self.deadImage.transpose(Image.FLIP_LEFT_RIGHT)
        #image = self.mode.scaleImage(image,0.4)
    
    def throwMove(self):
        #image from http://www.aigei.com/
        self.throwImage = self.mode.loadImage('player1throw.png')
        self.throwImageTrans = self.throwImage.transpose(Image.FLIP_LEFT_RIGHT)

    def hpImage(self):
        self.hps = []
        self.hpsSizes = dict()
        for i in range(0,10):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'playerhp{i}.png')
            image = self.mode.scaleImage(image,1.53)
            self.hps.append(image)
            (width, height) = image.size
            # get size of every getAttack
            self.hpsSizes[i] = (width, height)

    def mpImage(self):
        self.mps = []
        self.mpsSizes = dict()
        for i in range(0,10):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'playermp{i}.png')
            image = self.mode.scaleImage(image,1.53)
            self.mps.append(image)
            (width, height) = image.size
            # get size of every getAttack
            self.mpsSizes[i] = (width, height)

    def buyHpMedicine(self):
        if self.coins >= 100:
            self.mode.hpMedicine.hpNumber += 1
            self.coins -= 100
            #print(self.coins)
    
    def buyMpMedicine(self):
        if self.coins >= 100:
            self.mode.mpMedicine.mpNumber += 1
            self.coins -= 100
            #print(self.coins)
    
    def useHpMedicine(self):
        if self.mode.hpMedicine.hpNumber > 0:
            self.mode.hpMedicine.hpNumber -= 1
            self.hp += self.fullHp//2
            if self.hp > self.fullHp:
                self.hp = self.fullHp
    
    def useMpMedicine(self):
        if self.mode.mpMedicine.mpNumber > 0:
            self.mode.mpMedicine.mpNumber -= 1
            self.mp += self.fullMp//2
            if self.mp > self.fullMp:
                self.mp = self.fullMp

    def draw(self,canvas):
        sprite = self.sprites[self.mode.spriteCounter]
        spriteTrans = sprite.transpose(Image.FLIP_LEFT_RIGHT)
        spriteAttack = self.attackSprites[self.mode.spriteCounterAttack]
        spriteAttackTrans = spriteAttack.transpose(Image.FLIP_LEFT_RIGHT)
        spriteGetAttack = \
            self.getAttackSprites[self.mode.spriteCounterGetAttack]
        spriteGetAttackTrans = \
            spriteGetAttack.transpose(Image.FLIP_LEFT_RIGHT)
        throwTrans = self.throwImage.transpose(Image.FLIP_LEFT_RIGHT)
        attackMixHeight = self.y + self.attackSizes[188][1]//2
        attackMixWidth = self.x - self.attackSizes[188][0]//2
        getAttackMixHeight = self.y + self.getAttackSizes[1][1]//2
        hpsMixHeight = self.mode.height//10*9 + self.hpsSizes[0][1]//2
        mpsMixHeight = self.mode.height//10*9 + self.mpsSizes[0][1]//2
        # Fixed ordinate of character attack
        attackY = attackMixHeight -\
            self.attackSizes[188+self.mode.spriteCounterAttack][1]//2
        attackX = attackMixWidth +\
            self.attackSizes[188+self.mode.spriteCounterAttack][0]//2
        getAttackY = getAttackMixHeight -\
            self.getAttackSizes[1+self.mode.spriteCounterGetAttack][1]//2
        # draw by situation
        if self.status == self.stand and self.right == True:
            canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(self.player1Stand))
        elif self.status == self.stand and self.right == False:
            canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(self.player1StandTrans))                  
        elif self.status == self.run and self.right == True:
            canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(sprite))
        elif self.status == self.run and self.right == False:
            canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(spriteTrans))
        elif self.status == self.attack and self.right == True:
            canvas.create_image(attackX,attackY,\
                            image=ImageTk.PhotoImage(spriteAttack))
        elif self.status == self.attack and self.right == False:
            canvas.create_image(attackX,attackY,\
                            image=ImageTk.PhotoImage(spriteAttackTrans))
        elif self.status == self.getAttack and self.right == True:
            canvas.create_image(self.x,getAttackY,\
                            image=ImageTk.PhotoImage(spriteGetAttack))
        elif self.status == self.getAttack and self.right == False:
            canvas.create_image(self.x,getAttackY,\
                            image=ImageTk.PhotoImage(spriteGetAttackTrans))
        elif self.status == self.dead and self.right == True:
            canvas.create_image(self.x,self.y+self.height//2,\
                            image=ImageTk.PhotoImage(self.deadImage))
        elif self.status == self.dead and self.right == False:
            canvas.create_image(self.x,self.y+self.height//2,\
                            image=ImageTk.PhotoImage(self.deadImageTrans))
        elif self.status == self.throw:
            canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(self.throwImage))
        elif self.status == self.skill3 and self.right == True:
            canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(self.throwImage))
        elif self.status == self.skill3 and self.right == False:
            canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(throwTrans))

        # draw hp
        hpsX = self.mode.width//10*4
        for i in range(10):
            if (1-(i+1)*0.1)*self.fullHp < self.hp <= (1-i*0.1)*self.fullHp:
                hpsY = hpsMixHeight - self.hpsSizes[i][1]//2
                canvas.create_image\
                    (hpsX,hpsY,image=ImageTk.PhotoImage(self.hps[i]))
        #draw mp
        mpsX = self.mode.width//10*6
        for i in range(10):
            if (1-(i+1)*0.1)*self.fullMp < self.mp <= (1-i*0.1)*self.fullMp:
                mpsY = mpsMixHeight - self.mpsSizes[i][1]//2
                canvas.create_image\
                    (mpsX,mpsY,image=ImageTk.PhotoImage(self.mps[i]))
        
        

class player2(player1):
    def standAndRun(self):
        urlStand = 'nv90.png' #image from http://www.aigei.com/
        self.player1Stand = self.mode.loadImage(urlStand)
        self.player1StandTrans = self.mode.loadImage(urlStand).\
                                transpose(Image.FLIP_LEFT_RIGHT)
        self.sprites = []
        self.spritesTanspose = []
        for i in range(105,113):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'nv{i}.png') 
            self.sprites.append(image)
            self.spritesTanspose.append(image.transpose(Image.FLIP_LEFT_RIGHT))

    def moveInLevel2(self,direction):
        if direction == 'a': 
            self.x -= self.dx
            if self.x < 0:
                self.x = 0
        if direction == 'd': 
            self.x += self.dx
            if self.x > self.mode.width:
                self.x = self.mode.width
        if direction == 'w' and self.status != self.throw:
            self.y -= self.dy
            if self.y + self.height//2 < self.mode.limitY1:
                self.y = self.mode.limitY1 - self.height//2
        if direction == 's' and self.status != self.throw:
            self.y += self.dy
            if self.y + self.height//2 > self.mode.limitY2:
                self.y = self.mode.limitY2 - self.height//2

    def attackMove(self):
        self.attackSprites = []
        self.attackSizes = dict()
        for i in range(188,210):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'nv{i}.png') 
            self.attackSprites.append(image)
            (width, height) = image.size
            self.attackSizes[i] = (width, height)# get size of every attack
    
    def dieMove(self):
        #image from http://www.aigei.com/
        self.deadImage = self.mode.loadImage('nvgetattack9.png')
        self.deadImageTrans = self.deadImage.transpose(Image.FLIP_LEFT_RIGHT)
        #image = self.mode.scaleImage(image,0.4)
    
    def throwMove(self):
        #image from http://www.aigei.com/
        self.throwImage = self.mode.loadImage('player2throw.png')
        self.throwImageTrans = self.throwImage.transpose(Image.FLIP_LEFT_RIGHT)

    def getAttackMove(self):
        self.getAttackSprites = []
        self.getAttackSizes = dict()
        for i in range(1,11):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'nvgetattack{i}.png') 
            self.getAttackSprites.append(image)
            (width, height) = image.size
            # get size of every getAttack
            self.getAttackSizes[i] = (width, height)


class enemy1(object):
    def __init__(self,mode):
        self.mode = mode
        self.kind = 1
        self.enemy1Sprites = []
        self.enemy1AttackSprites = []
        self.enemy1GetAttackSprites = []
        self.getAttackSizes = dict()
        for i in range(21,25):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'enemy{i}.png') 
            image = self.mode.scaleImage(image,0.7)
            self.enemy1Sprites.append(image)
        for i in range(1,7):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'enemy2{i}{i}.png') 
            image = self.mode.scaleImage(image,0.7)
            self.enemy1AttackSprites.append(image)
        for i in range(1,8):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'enemy2{i}{i}{i}.png') 
            image = self.mode.scaleImage(image,0.7)
            self.enemy1GetAttackSprites.append(image)
            (width, height) = image.size
            # get size of every getAttack
            self.getAttackSizes[i] = (width, height)
            #if i == 5:
                #for _ in range(10):
                    #self.enemy1GetAttackSprites.append(image)
        self.width, self.height = self.enemy1Sprites[0].size
        #create enemy from left or right
        self.x = random.choice([(mode.width - 50), 50])
        self.y = random.randint(self.mode.limitY1 - self.height//2,\
                                self.mode.limitY2 - self.height//2)
        self.ds = 10
        self.dsToBase = 20
        self.attackDistance = 100
        self.status = 0
        self.run = 0
        self.attack = 1
        self.getAttack = 2
        self.runToBase = 3
        self.attackBase = 4
        self.hp = 3000
        self.fullHp = 3000
        self.attackValue = 20
        self.hpImage()
        self.coinValue = 50
        self.scoreValue = 500
        self.fall = False
    
    def fallToGround(self):
        if self.status == self.getAttack and\
            self.mode.spriteCounterEnemy1GetAttack ==\
            len(self.enemy1GetAttackSprites) - 1:
            self.fall = True
        elif self.status != self.getAttack:
            self.fall = False
    
    #def __hash__(self):
        #return hash(self.kind)

    #def __eq__(self, other):
        #return (isinstance(other, enemy1) and (self.kind == other.kind))
    
    def move(self):
        s = self.getDistance()
        sBase = self.getDistanceToBase()
        if self.mode.player1.status == self.mode.player1.dead or\
            self.mode.player2.status == self.mode.player1.dead:
            cosAngle = (self.x - self.mode.base.x) / sBase
            sinAngle = (self.mode.base.y+50-self.y) / sBase
            dx = - self.dsToBase * cosAngle
            dy = self.dsToBase * sinAngle
            if self.status == self.runToBase:
                self.x += dx
                self.y += dy
        else:
            if self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player1Index:
                cosAngle = (self.x - self.mode.player1.x) / s
                sinAngle = (self.mode.player1.y-self.y) / s
            if self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player2Index:
                cosAngle = (self.x - self.mode.player2.x) / s
                sinAngle = (self.mode.player2.y-self.y) / s
            dx = - self.ds * cosAngle
            dy = self.ds * sinAngle
            if self.status != self.getAttack:
                self.x += dx
                self.y += dy
    
    def getDistance(self):
        if self.mode.app.splashScreenMode.playerChooseIndex ==\
            self.mode.app.splashScreenMode.player1Index:
            return ((self.mode.player1.x - self.x) ** 2 + \
                (self.mode.player1.y - self.y) ** 2) ** 0.5
        if self.mode.app.splashScreenMode.playerChooseIndex ==\
            self.mode.app.splashScreenMode.player2Index:
            return ((self.mode.player2.x - self.x) ** 2 + \
                (self.mode.player2.y - self.y) ** 2) ** 0.5

    def getDistanceToBase(self):
        return ((self.mode.base.x - self.x) ** 2 + \
                (self.mode.base.y - self.y) ** 2) ** 0.5
    
    def getDistanceToSkill2(self):
        return ((self.mode.skill2.x - self.x) ** 2 + \
                (self.mode.skill2.y - self.y) ** 2) ** 0.5
    
    def getStatus(self):
        if self.mode.player1.status == self.mode.player1.dead or\
            self.mode.player2.status == self.mode.player2.dead:
            #assert enemy1 get attacked by base
            if self.getDistanceToBase() <= self.mode.base.attackDistance:
                self.hp -= self.mode.base.attackValue
                # lose if base hp is 0
                if self.mode.base.hp <= 0:
                    self.mode.app.setActiveMode(self.mode.app.loseMode)
            if self.getDistanceToBase() > self.attackDistance:
                self.status = self.runToBase
            else:
                self.status = self.attackBase
                self.mode.base.hp -= self.attackValue
        else:
            #assert enemy1 get attacked
            if self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player1Index:
                if self.getDistance() <= self.mode.player1.attackDistance and\
                    self.mode.player1.status == self.mode.player1.attack:
                    if ((self.mode.player1.x - self.x) <= 0 and\
                        self.mode.player1.right == True):
                        self.status = self.getAttack
                        self.x += 3
                        self.hp -= self.mode.player1.attackValue
                    elif ((self.mode.player1.x - self.x) >= 0 and\
                        self.mode.player1.right == False):
                        self.status = self.getAttack
                        self.x -= 3
                        self.hp -= self.mode.player1.attackValue
                elif self.getDistance() <= self.mode.player1.attackDistance and\
                    self.mode.player1.status == self.mode.player1.skill1:
                    self.status = self.getAttack
                    self.hp -= self.mode.skill1.attackValue
                elif self.mode.player1.status == self.mode.player1.skill3:
                    self.status = self.getAttack
                    self.hp -= self.mode.skill3.attackValue
                #enemy1 run
                elif self.getDistance() > self.attackDistance:
                    self.status = self.run
                #enemy1 attack
                else:
                    self.status = self.attack
            elif self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player2Index:
                if self.getDistance() <= self.mode.player2.attackDistance and\
                    self.mode.player2.status == self.mode.player2.attack:
                    if ((self.mode.player2.x - self.x) <= 0 and\
                        self.mode.player2.right == True):
                        self.status = self.getAttack
                        self.x += 3
                        self.hp -= self.mode.player2.attackValue
                    elif ((self.mode.player2.x - self.x) >= 0 and\
                        self.mode.player2.right == False):
                        self.status = self.getAttack
                        self.x -= 3
                        self.hp -= self.mode.player2.attackValue
                elif self.getDistance() <= self.mode.player2.attackDistance and\
                    self.mode.player2.status == self.mode.player2.skill1:
                    self.status = self.getAttack
                    self.hp -= self.mode.skill1.attackValue
                elif self.mode.player2.status == self.mode.player2.skill3:
                    self.status = self.getAttack
                    self.hp -= self.mode.skill3.attackValue
                #enemy1 run
                elif self.getDistance() > self.attackDistance:
                    self.status = self.run
                #enemy1 attack
                else:
                    self.status = self.attack
            if self.mode.skill2.status:
                if self.getDistanceToSkill2() <=\
                    self.mode.skill2.attackDistance:
                    self.status = self.getAttack
                    self.hp -= self.mode.skill2.attackValue
                    if self.mode.skill2.right:
                        self.x += 3
                    else:
                        self.x -= 3

    def hpImage(self):
        self.hps = []
        self.hpsSizes = dict()
        for i in range(0,5):
            #image from https://dnf.qq.com/
            image = self.mode.loadImage(f'playerFull{i}.png')
            image = self.mode.scaleImage(image,0.4)
            self.hps.append(image)
            (width, height) = image.size
            # get size of every getAttack
            self.hpsSizes[i] = (width, height)

    def draw(self,canvas):
        sprite = self.enemy1Sprites[self.mode.spriteCounterEnemy1]
        spriteTrans = sprite.transpose(Image.FLIP_LEFT_RIGHT)
        attackSprite = self.enemy1AttackSprites\
                       [self.mode.spriteCounterEnemy1Attack]
        attackSpriteTrans = attackSprite.transpose(Image.FLIP_LEFT_RIGHT)
        getAttackSprite = self.enemy1GetAttackSprites\
                        [self.mode.spriteCounterEnemy1GetAttack]
        getAttackSpriteTrans = getAttackSprite.transpose(Image.FLIP_LEFT_RIGHT)
        getAttackFall = self.enemy1GetAttackSprites\
                        [len(self.enemy1GetAttackSprites) - 2]
        getAttackFallTrans = getAttackFall.transpose(Image.FLIP_LEFT_RIGHT)
        getAttackMixHeight = self.y + self.getAttackSizes[1][1]//2
        getAttackY = getAttackMixHeight -\
            self.getAttackSizes[1+self.mode.spriteCounterEnemy1GetAttack][1]//2
        getAttackFallY = getAttackMixHeight -\
            self.getAttackSizes[1+len(self.enemy1GetAttackSprites) - 3][1]//2
        #draw enemy by position to player or base
        if self.mode.player1.status == self.mode.player1.dead or\
            self.mode.player2.status == self.mode.player1.dead:
            if self.status == self.runToBase:
                if (self.mode.base.x - self.x) <= 0:
                            canvas.create_image(self.x,self.y,\
                                        image=ImageTk.PhotoImage(spriteTrans))
                else:
                            canvas.create_image(self.x,self.y,\
                                        image=ImageTk.PhotoImage(sprite))
            if self.status == self.attackBase:
                if (self.mode.base.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSpriteTrans))
                else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSprite))

        else:
            if self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player1Index:
                if self.status == self.getAttack:
                    if (self.mode.player1.x - self.x) <= 0:
                        if self.fall == False:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackSpriteTrans))
                        else:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackFallTrans))
                    else:
                        if self.fall == False:
                            canvas.create_image(self.x,getAttackFallY,\
                                    image=ImageTk.PhotoImage(getAttackSprite))
                        else:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackFall))
                elif self.status == self.run:
                    if (self.mode.player1.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(spriteTrans))
                    else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(sprite))
                elif self.status == self.attack:
                    if (self.mode.player1.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSpriteTrans))
                    else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSprite))
                
            elif self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player2Index:
                if self.status == self.getAttack:
                    if (self.mode.player2.x - self.x) <= 0:
                        if self.fall == False:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackSpriteTrans))
                        else:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackFallTrans))
                    else:
                        if self.fall == False:
                            canvas.create_image(self.x,getAttackFallY,\
                                    image=ImageTk.PhotoImage(getAttackSprite))
                        else:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackFall))
                elif self.status == self.run:
                    if (self.mode.player2.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(spriteTrans))
                    else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(sprite))
                elif self.status == self.attack:
                    if (self.mode.player2.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSpriteTrans))
                    else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSprite))
                
        #draw enemy hp
        for i in range(5):
            if (1-(i+1)*0.2)*self.fullHp < self.hp <= (1-i*0.2)*self.fullHp:
                canvas.create_image\
                (self.x,self.y-self.height//2-30,\
                image=ImageTk.PhotoImage(self.hps[i]))
                break
        


class enemy2(enemy1):
    def __init__(self,mode):
        self.mode = mode
        self.kind = 2
        self.enemy2Sprites = []
        self.enemy2AttackSprites = []
        self.enemy2GetAttackSprites = []
        self.getAttackSizes = dict()
        for i in range(1,7):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'enemy3{i}.png') 
            image = self.mode.scaleImage(image,0.7)
            self.enemy2Sprites.append(image)
        for i in range(7,21):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'enemy3{i}.png') 
            image = self.mode.scaleImage(image,0.7)
            self.enemy2AttackSprites.append(image)
        for i in range(21,27):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'enemy3{i}.png') 
            image = self.mode.scaleImage(image,0.7)
            self.enemy2GetAttackSprites.append(image)
            (width, height) = image.size
            # get size of every getAttack
            self.getAttackSizes[i] = (width, height)
            #if i == 25:
                #for _ in range(10):
                    #self.enemy2GetAttackSprites.append(image)
        self.width, self.height = self.enemy2Sprites[0].size
        #create enemy from left or right
        self.x = random.choice([(mode.width - 50), 50])
        self.y = random.randint(self.mode.limitY1 - self.height//2,\
                                self.mode.limitY2 - self.height//2)
        self.ds = 10
        self.dsToBase = 20
        self.attackDistance = 90
        self.status = 0
        self.run = 0
        self.attack = 1
        self.getAttack = 2
        self.runToBase = 3
        self.attackBase = 4
        self.hp = 5000
        self.fullHp = 5000
        self.attackValue = 30
        self.hpImage()
        self.coinValue = 70
        self.scoreValue = 1000
        self.fall = False
    
    def fallToGround(self):
        if self.status == self.getAttack and\
            self.mode.spriteCounterEnemy2GetAttack ==\
            len(self.enemy2GetAttackSprites) - 1:
            self.fall = True
        elif self.status != self.getAttack:
            self.fall = False

    def draw(self,canvas):
        sprite = self.enemy2Sprites[self.mode.spriteCounterEnemy2]
        spriteTrans = sprite.transpose(Image.FLIP_LEFT_RIGHT)
        attackSprite = self.enemy2AttackSprites\
                       [self.mode.spriteCounterEnemy2Attack]
        attackSpriteTrans = attackSprite.transpose(Image.FLIP_LEFT_RIGHT)
        getAttackSprite = self.enemy2GetAttackSprites\
                        [self.mode.spriteCounterEnemy2GetAttack]
        getAttackFall = self.enemy2GetAttackSprites\
                        [len(self.enemy2GetAttackSprites) - 2]
        getAttackFallTrans = getAttackFall.transpose(Image.FLIP_LEFT_RIGHT)
        getAttackSpriteTrans = getAttackSprite.transpose(Image.FLIP_LEFT_RIGHT)
        getAttackMixHeight = self.y + self.getAttackSizes[21][1]//2
        getAttackY = getAttackMixHeight -\
            self.getAttackSizes[21+self.mode.spriteCounterEnemy2GetAttack][1]//2
        getAttackFallY = getAttackMixHeight -\
            self.getAttackSizes[21+len(self.enemy2GetAttackSprites) - 2][1]//2
        #draw enemy by position to player or base
        if self.mode.player1.status == self.mode.player1.dead or\
            self.mode.player2.status == self.mode.player1.dead:
            if self.status == self.runToBase:
                if (self.mode.base.x - self.x) <= 0:
                            canvas.create_image(self.x,self.y,\
                                        image=ImageTk.PhotoImage(spriteTrans))
                else:
                            canvas.create_image(self.x,self.y,\
                                        image=ImageTk.PhotoImage(sprite))
            if self.status == self.attackBase:
                if (self.mode.base.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSpriteTrans))
                else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSprite))

        else:
            if self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player1Index:
                if self.status == self.run:
                    if (self.mode.player1.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(spriteTrans))
                    else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(sprite))
                elif self.status == self.attack:
                    if (self.mode.player1.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSpriteTrans))
                    else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSprite))
                elif self.status == self.getAttack:
                    if (self.mode.player1.x - self.x) <= 0:
                        if self.fall == False:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackSpriteTrans))
                        else:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackFallTrans))
                    else:
                        if self.fall == False:
                            canvas.create_image(self.x,getAttackFallY,\
                                    image=ImageTk.PhotoImage(getAttackSprite))
                        else:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackFall))
            elif self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player2Index:
                if self.status == self.run:
                    if (self.mode.player2.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(spriteTrans))
                    else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(sprite))
                elif self.status == self.attack:
                    if (self.mode.player2.x - self.x) <= 0:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSpriteTrans))
                    else:
                        canvas.create_image(self.x,self.y,\
                                    image=ImageTk.PhotoImage(attackSprite))
                elif self.status == self.getAttack:
                    if (self.mode.player2.x - self.x) <= 0:
                        if self.fall == False:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackSpriteTrans))
                        else:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackFallTrans))
                    else:
                        if self.fall == False:
                            canvas.create_image(self.x,getAttackFallY,\
                                    image=ImageTk.PhotoImage(getAttackSprite))
                        else:
                            canvas.create_image(self.x,getAttackFallY,\
                                image=ImageTk.PhotoImage(getAttackFall))
        #draw enemy hp
        for i in range(5):
            if (1-(i+1)*0.2)*self.fullHp < self.hp <= (1-i*0.2)*self.fullHp:
                canvas.create_image\
                (self.x,self.y-self.height//2-30,\
                image=ImageTk.PhotoImage(self.hps[i]))
                break


class enemy3(enemy1):
    def __init__(self,mode):
        self.mode = mode
        self.x = self.mode.width -100
        self.y = 100
        self.kind = 3
        self.enemy3Sprites = []
        self.enemy3AttackSprites = []
        self.enemy3IceSprites = []
        self.enemy3GetAttackSprites = []
        self.attackInterval = 10
        #image from http://www.aigei.com/
        self.standImage = self.mode.loadImage(f'enemy41.png') 
        #image from http://www.aigei.com/
        freeze = self.mode.loadImage(f'freeze.png') 
        self.freeze = self.mode.scaleImage(freeze,0.35)
        for i in range(2,8):
            if i == 2:
                for _ in range(self.attackInterval):
                    self.enemy3AttackSprites.append(self.standImage)
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'enemy4{i}.png') 
            image = self.mode.scaleImage(image,1)
            self.enemy3AttackSprites.append(image)
        for i in range(8,14):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'enemy4{i}.png') 
            image = self.mode.scaleImage(image,1)
            self.enemy3GetAttackSprites.append(image)
            if i == 12:
                for _ in range(self.attackInterval):
                    self.enemy3GetAttackSprites.append(image)
        for i in range(14,22):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'enemy4{i}.png') 
            image = self.mode.scaleImage(image,1)
            self.enemy3Sprites.append(image)
        for i in range(0,5):
            #image from http://www.aigei.com/
            iceImage = self.mode.loadImage(f'ice{i}.png')
            iceImage = self.mode.scaleImage(iceImage,1.2)
            self.enemy3IceSprites.append(iceImage)
        self.width, self.height = self.enemy3Sprites[0].size
        self.status = 0
        self.run = 0
        self.attack = 1
        self.getAttack = 2
        self.runToBase = 3
        self.attackBase = 4
        self.frozen = 5
        self.frozenTimer = 0
        self.hp = 10000
        self.fullHp = 10000
        self.hpImage()
        self.coinValue = 70
        self.scoreValue = 1000
        self.attackValue = 100
        self.dy = 10

    def getDistanceToExplosive(self):
        return ((self.x-self.mode.explosive.x)**2 +\
                (self.y-self.mode.explosive.y)**2) ** 0.5
    
    def getDistanceToExplosive2(self):
        return ((self.x-self.mode.explosive2.x)**2 +\
                (self.y-self.mode.explosive2.y)**2) ** 0.5

    def getAttackInfrozen(self):
        if self.getDistanceToExplosive() < self.mode.explosive.attackDistance:
            self.mode.explosive.boom = True
            self.hp -= self.mode.explosive.attackValue
            if self.hp <= 0:
                self.mode.app.setActiveMode(self.mode.app.winMode)
        elif self.getDistanceToExplosive2()< self.mode.explosive.attackDistance:
            self.mode.explosive2.boom = True
            self.hp -= self.mode.explosive2.attackValue
            if self.hp <= 0:
                self.mode.app.setActiveMode(self.mode.app.winMode)

    def getStatus(self):
        if self.mode.player1.status == self.mode.player1.dead or\
            self.mode.player2.status == self.mode.player2.dead:
            #assert enemy1 get attacked by base
            self.status = self.attackBase
            self.mode.base2.hp -= self.attackValue / 10
            if self.mode.base2.hp <= 0:
                self.mode.app.setActiveMode(self.mode.app.loseMode)
        elif self.getDistanceToExplosive() < self.mode.explosive.attackDistance:
            self.mode.explosive.boom = True
            self.status = self.getAttack
            self.hp -= self.mode.explosive.attackValue
            if self.hp <= 0:
                self.mode.app.setActiveMode(self.mode.app.winMode)
        elif self.getDistanceToExplosive2()< self.mode.explosive.attackDistance:
            self.mode.explosive2.boom = True
            self.status = self.getAttack
            self.hp -= self.mode.explosive2.attackValue
            if self.hp <= 0:
                self.mode.app.setActiveMode(self.mode.app.winMode)
        #assert run or attack
        elif self.mode.runAndAttackCount // 30 % 2 == 0:
            self.status = self.run
        else:
            self.status = self.attack
    
    def attackPlayerOnce(self):
        #attack player when ice break but didn't make player get into
        #getAttack status since it will stop the throw
        IceBreakIndex = 2
        if self.mode.spriteCounterEnemy3Attack -\
                        self.attackInterval -1 == IceBreakIndex:
            if self.status == self.attack:
                self.mode.player1.hp -= self.attackValue
                self.mode.player2.hp -= self.attackValue
    
    def getRidOfFrozen(self):
        if self.status == self.frozen:
            self.frozenTimer += 1
            if self.frozenTimer >= 30:
                self.status = self.run
                self.frozenTimer = 0
            
    
    def fly(self):
        if self.status == self.run:
            self.y += self.dy
        if self.y >= self.mode.height//2 or\
            self.y - self.height//2 <= 0:
            self.dy = -self.dy

    def draw(self,canvas):
        sprite = self.enemy3Sprites[self.mode.spriteCounterEnemy3]
        spriteTrans = sprite.transpose(Image.FLIP_LEFT_RIGHT)
        attackSprite = self.enemy3AttackSprites\
                       [self.mode.spriteCounterEnemy3Attack].\
                       transpose(Image.FLIP_LEFT_RIGHT)
        getAttackSprite = self.enemy3GetAttackSprites\
                        [self.mode.spriteCounterEnemy3GetAttack]
        getAttackSpriteTrans = getAttackSprite.transpose(Image.FLIP_LEFT_RIGHT)
        if self.mode.spriteCounterEnemy3Attack > self.attackInterval:
            iceSprite = self.enemy3IceSprites\
                        [self.mode.spriteCounterEnemy3Attack -\
                         self.attackInterval -1]
        if self.status == self.run:
            canvas.create_image(self.x,self.y,\
                                image=ImageTk.PhotoImage(spriteTrans))
        elif self.status == self.attack:
        #or self.status == self.attackBase:
            canvas.create_image(self.x,self.y,\
                                image=ImageTk.PhotoImage(attackSprite))
        elif self.status == self.attackBase:
        #or self.status == self.attackBase:
            canvas.create_image(self.x,self.y,\
                                image=ImageTk.PhotoImage(attackSprite))
        elif self.status == self.getAttack:
            canvas.create_image(self.x,self.y,\
                                image=ImageTk.PhotoImage(getAttackSpriteTrans))
        elif self.status == self.frozen:
            canvas.create_image(self.x,self.y,\
                                image=ImageTk.PhotoImage\
                                (self.standImage.\
                                transpose(Image.FLIP_LEFT_RIGHT)))
            canvas.create_image(self.x+10,self.y-10,\
                                image=ImageTk.PhotoImage(self.freeze))
        if self.mode.spriteCounterEnemy3Attack > self.attackInterval:
            if self.status == self.attack:
                canvas.create_image(self.mode.player1.x,\
                    self.mode.player1.y,image=ImageTk.PhotoImage(iceSprite))
                canvas.create_image(self.mode.player2.x,\
                    self.mode.player2.y,image=ImageTk.PhotoImage(iceSprite))
            elif self.status == self.attackBase:
                canvas.create_image(self.mode.base2.x,\
                    self.mode.base2.y,image=ImageTk.PhotoImage(iceSprite))
        #draw hp
        for i in range(5):
            if (1-(i+1)*0.2)*self.fullHp < self.hp <= (1-i*0.2)*self.fullHp:
                canvas.create_image\
                (self.x,self.y-self.height//2-30,\
                image=ImageTk.PhotoImage(self.hps[i]))
                break


class hpMedicine(object):
    def __init__(self,mode):
        self.mode = mode
        #self.hpImage1 = self.mode.loadImage('hpmedicine1.png')
        self.hpImage2 = self.mode.loadImage('hpmedicine2.png')
        self.hpImage2 = self.mode.scaleImage(self.hpImage2,1.35)
        self.hpNumber = 0
    
    def draw(self,canvas):
        canvas.create_image(95,self.mode.height-23,\
                            image=ImageTk.PhotoImage(self.hpImage2))
        canvas.create_text(85,self.mode.height-33,fill = 'white',\
                        text=f'{self.hpNumber}',font=f'Arial 10 bold')

class mpMedicine(object):
    def __init__(self,mode):
        self.mode = mode
        #self.mpImage1 = self.mode.loadImage('hpmedicine1.png')
        mpImage2 = self.mode.loadImage('mpmedicine2.png')
        self.mpImage2 = self.mode.scaleImage(mpImage2,1.35)
        self.mpNumber = 0
    
    def draw(self,canvas):
        canvas.create_image(137,self.mode.height-23,\
                            image=ImageTk.PhotoImage(self.mpImage2))
        canvas.create_text(127,self.mode.height-33,fill = 'white',\
                        text=f'{self.mpNumber}',font=f'Arial 10 bold')


class skill(object):
    def __init__(self,mode):
        self.mode = mode
        #image from https://dnf.qq.com/
        self.tenmpSkillBar = self.mode.loadImage('skill.jpg')  
        self.skillBar = self.mode.scaleImage(self.tenmpSkillBar,1.3)
        self.width, self.height = self.skillBar.size
        #image from https://dnf.qq.com/
        self.skillBar1 = mode.scaleImage\
                        (self.mode.loadImage('skillBar1.jpg'),1.1)
        #image from https://dnf.qq.com/
        self.skillBar2 = mode.scaleImage\
                        (self.mode.loadImage('skillBar2.png'),0.28)
        #image from https://dnf.qq.com/
        self.skillBar3 = mode.scaleImage\
                        (self.mode.loadImage('skillBar3.png'),0.19)
        #image from https://dnf.qq.com/
        self.hpframe = mode.scaleImage\
                        (self.mode.loadImage('hpframe.png'),1.3)
    
    def draw(self,canvas):
        hpsX = self.mode.width//10*4
        mpsX = self.mode.width//10*6+1
        canvas.create_image(200,self.mode.height-25,\
                            image=ImageTk.PhotoImage(self.skillBar))
        canvas.create_image(self.mode.width-200,self.mode.height-25,\
                            image=ImageTk.PhotoImage(self.skillBar))
        canvas.create_image(self.mode.width-180-self.width//2,\
                            self.mode.height-25,\
                            image=ImageTk.PhotoImage(self.skillBar1))
        canvas.create_image(self.mode.width-140-self.width//2,\
                            self.mode.height-25,\
                            image=ImageTk.PhotoImage(self.skillBar2))
        canvas.create_image(self.mode.width-99-self.width//2,\
                            self.mode.height-25,\
                            image=ImageTk.PhotoImage(self.skillBar3))
        #draw hpframe
        canvas.create_image(hpsX,self.mode.height-48,\
                            image=ImageTk.PhotoImage(self.hpframe))
        #draw mpframe
        canvas.create_image(mpsX,self.mode.height-48,\
                            image=ImageTk.PhotoImage(self.hpframe))
    
class skillInLevel2(skill):
    def draw(self,canvas):
        hpsX = self.mode.width//10*4
        mpsX = self.mode.width//10*6+1
        #draw hpframe
        canvas.create_image(hpsX,self.mode.height-48,\
                            image=ImageTk.PhotoImage(self.hpframe))
        #draw mpframe
        canvas.create_image(mpsX,self.mode.height-48,\
                            image=ImageTk.PhotoImage(self.hpframe))


class skill1(object):
    def __init__(self,mode):
        self.mode = mode
        self.attackValue = 200
        self.skill1Sprites = []
        self.skill1Sizes = dict()
        for i in range(1,8):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'skill1{i}.png')
            image = self.mode.scaleImage(image,0.7)
            self.skill1Sprites.append(image)
            if i ==7:
                self.skill1Sprites.append(image)
            (width, height) = image.size
            self.skill1Sizes[i] = (width, height)
        #image from http://www.aigei.com/
        self.player1skill1 = self.mode.loadImage('player1skill1.png')
        #image from http://www.aigei.com/
        self.player2skill1 = self.mode.loadImage('player2skill1.png')
        self.player1skill1Trans = \
            self.player1skill1.transpose(Image.FLIP_LEFT_RIGHT)
        self.player2skill1Trans = \
            self.player2skill1.transpose(Image.FLIP_LEFT_RIGHT)
        

    def stopSkill1(self):
        if self.mode.spriteCounterSkill1 == len(self.skill1Sprites)-1:
                self.mode.player1.status = 0
                self.mode.player2.status = 0
                #self.mode.spriteCounterSkill1 = 0
    
    def draw(self,canvas):
        sprite = self.skill1Sprites[self.mode.spriteCounterSkill1]
        # get skill1 mixed
        if self.mode.player1.status == self.mode.player1.skill1:
            skill1MixHeight = self.mode.player1.y + self.skill1Sizes[1][1]//2
            skill1Y = skill1MixHeight -\
                self.skill1Sizes[1+self.mode.spriteCounterSkill1][1]//2 + 30
            if self.mode.player1.right == True:
                canvas.create_image(self.mode.player1.x,self.mode.player1.y,\
                            image=ImageTk.PhotoImage(self.player1skill1))
            else:
                canvas.create_image(self.mode.player1.x,self.mode.player1.y,\
                            image=ImageTk.PhotoImage(self.player1skill1Trans))
            canvas.create_image(self.mode.player1.x,skill1Y,\
                            image=ImageTk.PhotoImage(sprite))
        elif self.mode.player2.status == self.mode.player2.skill1:
            skill1MixHeight = self.mode.player2.y + self.skill1Sizes[1][1]//2
            skill1Y = skill1MixHeight -\
                self.skill1Sizes[1+self.mode.spriteCounterSkill1][1]//2 + 30
            if self.mode.player2.right == True:
                canvas.create_image(self.mode.player2.x,self.mode.player2.y,\
                            image=ImageTk.PhotoImage(self.player2skill1))
            else:
                canvas.create_image(self.mode.player2.x,self.mode.player2.y,\
                            image=ImageTk.PhotoImage(self.player2skill1Trans))
            canvas.create_image(self.mode.player2.x,skill1Y,\
                            image=ImageTk.PhotoImage(sprite))

class skill2(object):
    def __init__(self,mode):
        self.mode = mode
        self.x = 0
        self.y = 0
        self.attackValue = 300
        self.skill2Sprites = []
        self.status = False
        for i in range(1,7):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'skill2{i}.png')
            image = self.mode.scaleImage(image,0.6)
            self.skill2Sprites.append(image)
        self.skill2Sizes = image.size
        #image from http://www.aigei.com/
        self.player1skill2 = self.mode.loadImage('player1skill2.png')
        #image from http://www.aigei.com/
        self.player2skill2 = self.mode.loadImage('player2skill2.png')
        self.player1skill2Trans = \
            self.player1skill2.transpose(Image.FLIP_LEFT_RIGHT)
        self.player2skill2Trans = \
            self.player2skill2.transpose(Image.FLIP_LEFT_RIGHT)
        self.right = True
        self.attackDistance = 150
        #self.boom = False
            
    def skillMove(self):
        if self.status == True:
            if self.right == True:
                self.x += 30
            else:
                self.x -= 30
        else:
            if self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player1Index:
                self.x = self.mode.player1.x
                self.y = self.mode.player1.y
            else:
                self.x = self.mode.player2.x
                self.y = self.mode.player2.y
    
    def limit(self):
        return (0-self.skill2Sizes[0]//2 < self.x\
                < self.mode.width+self.skill2Sizes[0]//2) and\
                (0-self.skill2Sizes[1]//2 < self.y\
                < self.mode.height+self.skill2Sizes[1]//2)
    
    def resetPosition(self):
        if not ((0-self.skill2Sizes[0]//2 < self.x\
            < self.mode.width+self.skill2Sizes[0]//2) and\
            (0-self.skill2Sizes[1]//2 < self.y\
            < self.mode.height+self.skill2Sizes[1]//2)):
            self.status = False
            self.skillMove()
            
    
    def getDirection(self):
        if self.mode.app.splashScreenMode.playerChooseIndex ==\
            self.mode.app.splashScreenMode.player1Index:
            if self.mode.player1.right == True:
                self.right = True
            else:
                self.right = False
        elif self.mode.app.splashScreenMode.playerChooseIndex ==\
            self.mode.app.splashScreenMode.player2Index:
            if self.mode.player2.right == True:
                self.right = True
            else:
                self.right = False
    
    def draw(self,canvas):
        sprite = self.skill2Sprites[self.mode.spriteCounterSkill2]
        spriteTrans = sprite.transpose(Image.FLIP_LEFT_RIGHT)
        if self.mode.player1.status == self.mode.player1.skill2:
            if self.mode.player1.right == True:
                canvas.create_image(self.mode.player1.x,self.mode.player1.y,\
                            image=ImageTk.PhotoImage(self.player1skill2))
            else:
                canvas.create_image(self.mode.player1.x,self.mode.player1.y,\
                            image=ImageTk.PhotoImage(self.player1skill2Trans))
        elif self.mode.player2.status == self.mode.player2.skill2:
            if self.mode.player2.right == True:
                canvas.create_image(self.mode.player2.x,self.mode.player2.y,\
                            image=ImageTk.PhotoImage(self.player2skill2))
            else:
                canvas.create_image(self.mode.player2.x,self.mode.player2.y,\
                            image=ImageTk.PhotoImage(self.player2skill2Trans))
        if self.status == True and self.limit():
            if self.right == True:
                canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(sprite))
            else:
                canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(spriteTrans))

class skill3(skill1):
    def __init__(self,mode):
        self.mode = mode
        self.x = 0
        self.y = 0
        self.attackValue = 400
        self.skill3Sprites = []
        self.skill3PlayerSprites = []
        self.status = False
        for i in range(1,6):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'skill3{i}.png')
            image = self.mode.scaleImage(image,1)
            self.skill3Sprites.append(image)
            if i == 5:
                self.skill3Sprites.append(image)
        for i in range(1,10):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'skill3{i}{i}.png')
            image = self.mode.scaleImage(image,1.3)
            self.skill3PlayerSprites.append(image)

    def stopSkill3(self):
        if self.mode.spriteCounterSkill3 == len(self.skill3Sprites)-1:
                self.mode.player1.status = 0
                self.mode.player2.status = 0

    def draw(self,canvas):
        sprite = self.skill3Sprites[self.mode.spriteCounterSkill3]
        playerSprite = self.skill3PlayerSprites\
                        [self.mode.spriteCounterPlayerSkill3]
        playerSpriteTrans = playerSprite.transpose(Image.FLIP_LEFT_RIGHT)
        if self.mode.player1.status == self.mode.player1.skill3:
            for enemy in self.mode.enemySet:
                canvas.create_image(enemy.x,enemy.y,\
                            image=ImageTk.PhotoImage(sprite))
            if self.mode.player1.right == True:
                canvas.create_image(self.mode.player1.x + 80,\
                        self.mode.player1.y - 48,
                        image=ImageTk.PhotoImage(playerSprite))
            else:
                canvas.create_image(self.mode.player1.x - 80,\
                        self.mode.player1.y - 48,
                        image=ImageTk.PhotoImage(playerSpriteTrans))
        elif self.mode.player2.status == self.mode.player2.skill3:
            for enemy in self.mode.enemySet:
                canvas.create_image(enemy.x,enemy.y,\
                            image=ImageTk.PhotoImage(sprite))
            if self.mode.player2.right == True:
                canvas.create_image(self.mode.player2.x + 80,\
                        self.mode.player2.y - 48,
                        image=ImageTk.PhotoImage(playerSprite))
            else:
                canvas.create_image(self.mode.player2.x - 80,\
                        self.mode.player2.y - 48,
                        image=ImageTk.PhotoImage(playerSpriteTrans))


class explosive(object):
    def __init__(self,mode):
        self.mode = mode
        self.initPosition()
        #image from http://www.aigei.com/
        self.explosiveImage = self.mode.loadImage('explosive.png')
        self.explosiveSprites = []
        for i in range(0,13):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'boom{i}.png')
            image = self.mode.scaleImage(image,1)
            self.explosiveSprites.append(image)
        self.ups = 0
        self.launch = False
        self.angle = 0
        self.ds = 30
        self.dx = 25
        self.dy = 10
        self.dya = 1
        self.fallCount = 0   
        self.attackDistance = 50
        self.attackValue = 300
        self.boom = False
    
    def initPosition(self):
        self.x = self.mode.player1.x + 55
        self.y = self.mode.player1.y - 48
    
    def getPosition(self):
        if self.launch == False:
            self.x = self.mode.player1.x + 55
            self.y = self.mode.player1.y - 48
        elif not ((0 < self.x < self.mode.width) and\
                 (0 < self.y < self.mode.height)):
            self.x = self.mode.player1.x + 55
            self.y = self.mode.player1.y - 48
            self.mode.player1.stastus = self.mode.player1.stand
            self.ups = 0
            self.launch = False
            self.fallCount = 0 
        else:
            self.fallCount += 1
            self.x += self.ds*math.cos(self.angle)
            self.y -= self.ds*math.sin(self.angle) - self.fallCount*self.dya
    
    def stopBoom(self):
        if self.mode.spriteCounterExplosive1 == len(self.explosiveSprites)-1:
            self.boom = False
            self.initPosition()
            self.launch = False
            self.mode.spriteCounterExplosive1 = 0
    
    def drawLine(self,canvas):
        if self.launch == False:
            stopX = self.x + 100*math.cos(self.angle)
            stopY = self.y - 100*math.sin(self.angle)
            canvas.create_line(self.x,self.y,stopX,stopY,width = 2)

    def draw(self,canvas):
        explosiveSprite = self.explosiveSprites\
                        [self.mode.spriteCounterExplosive1]
        if self.boom == True:
            canvas.create_image(self.mode.enemy3.x,self.mode.enemy3.y,\
                        image=ImageTk.PhotoImage(explosiveSprite))
        if self.mode.player1.status == self.mode.player1.throw:
            self.drawLine(canvas)
            if self.boom == False:
                canvas.create_image(self.x,self.y,\
                        image=ImageTk.PhotoImage(self.explosiveImage))

class explosive2(explosive):
    def initPosition(self):
        self.x = self.mode.player2.x + 55
        self.y = self.mode.player2.y - 48

    def getPosition(self):
        if self.launch == False:
            self.x = self.mode.player2.x + 55
            self.y = self.mode.player2.y - 48
        elif not ((0 < self.x < self.mode.width) and\
                 (0 < self.y< self.mode.height)):
            self.x = self.mode.player2.x + 55
            self.y = self.mode.player2.y - 48
            self.mode.player2.stastus = self.mode.player2.stand
            self.ups = 0
            self.launch = False
            self.fallCount = 0 
        else:
            self.fallCount += 1
            self.x += self.ds*math.cos(self.angle)
            self.y -= self.ds*math.sin(self.angle) - self.fallCount*self.dya

    def stopBoom(self):
        if self.mode.spriteCounterExplosive2 == len(self.explosiveSprites)-1:
            self.boom = False
            self.initPosition()
            self.launch = False
            self.mode.spriteCounterExplosive2 = 0

    def draw(self,canvas):
        #print(self.boom)
        explosiveSprite = self.explosiveSprites\
                        [self.mode.spriteCounterExplosive2]
        if self.boom == True:
            canvas.create_image(self.mode.enemy3.x,self.mode.enemy3.y,\
                        image=ImageTk.PhotoImage(explosiveSprite))
        if self.mode.player2.status == self.mode.player2.throw:
            self.drawLine(canvas)
            if self.boom == False:
                canvas.create_image(self.x,self.y,\
                        image=ImageTk.PhotoImage(self.explosiveImage))


class store(object):
    def __init__(self,mode):
        self.mode = mode
        #image from http://www.aigei.com/
        self.storeImage = self.mode.loadImage('store.png')
        self.storeX = self.mode.width//5
        self.storeY = self.mode.height//2
        #image from http://www.aigei.com/
        hpImage2 = self.mode.loadImage('hpmedicine2.png')
        self.hpImage2 = self.mode.scaleImage(hpImage2,1.2)
        self.hpImage2Width, self.hpImage2Height = self.hpImage2.size
        #image from http://www.aigei.com/
        mpImage2 = self.mode.loadImage('mpmedicine2.png')
        self.mpImage2 = self.mode.scaleImage(mpImage2,1.2)
        self.mpImage2Width, self.mpImage2Height = self.mpImage2.size
        self.hpX = self.mode.width//5 - 103
        self.hpY = self.mode.height//2 -53
        self.mpX = self.mode.width//5 - 71
        self.mpY = self.mode.height//2 -53
        self.hpLeft = self.hpX - self.hpImage2Width//2
        self.hpRight = self.hpX + self.hpImage2Width//2
        self.hpTop = self.hpY - self.hpImage2Height//2
        self.hpDown = self.hpY + self.hpImage2Height//2
        self.mpLeft = self.mpX - self.mpImage2Width//2
        self.mpRight = self.mpX + self.mpImage2Width//2
        self.mpTop = self.mpY - self.mpImage2Height//2
        self.mpDown = self.mpY + self.mpImage2Height//2
        self.openStatus = False
    
    def draw(self,canvas):
        if self.openStatus == True:
            canvas.create_image(self.storeX,self.storeY,\
                            image=ImageTk.PhotoImage(self.storeImage))
            canvas.create_image(self.hpX,self.hpY,\
                            image=ImageTk.PhotoImage(self.hpImage2))
            canvas.create_image(self.mpX,self.mpY,\
                            image=ImageTk.PhotoImage(self.mpImage2))

class propertyPanel(object):
    def __init__(self,mode):
        self.mode = mode
        #image from http://www.aigei.com/
        panelImage = self.mode.loadImage('panel.png')
        self.panelImage = self.mode.scaleImage(panelImage,1.5)
        self.panelX = self.mode.width//5*3
        self.panelY = self.mode.height//2
        self.openStatus = False
    
    def draw(self,canvas):
        if self.openStatus == True:
            if self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player1Index:
                coins = self.mode.player1.coins
                attackValue = self.mode.player1.attackValue
                levels = self.mode.levels
                score = self.mode.score
                skill1AttackValue = self.mode.skill1.attackValue
                skill2AttackValue = self.mode.skill2.attackValue
                canvas.create_image(self.panelX,self.panelY,\
                            image=ImageTk.PhotoImage(self.panelImage))
                canvas.create_text(self.panelX-130,self.panelY-100,\
                    fill = 'black',text=f'Coins:  {coins}',\
                    font=f'Arial 17 bold', anchor='nw')
                canvas.create_text(self.panelX-130,self.panelY-60,\
                    fill = 'black', anchor='nw',\
                    text=f'AttackValue:  {attackValue}',font=f'Arial 17 bold')
                canvas.create_text(self.panelX-130,self.panelY-20,\
                    fill = 'black', anchor='nw',\
                    text=f'Skill1AttackValue:  {skill1AttackValue}',\
                    font=f'Arial 17 bold')
                canvas.create_text(self.panelX-130,self.panelY+20,\
                    fill = 'black', anchor='nw',\
                    text=f'Skill2AttackValue:  {skill2AttackValue}',\
                    font=f'Arial 17 bold')
                canvas.create_text(self.panelX-130,self.panelY+60,\
                    fill = 'black', anchor='nw',\
                    text=f'Levels:  {levels}',font=f'Arial 17 bold')
                canvas.create_text(self.panelX-130,self.panelY+100,\
                    fill = 'black', anchor='nw',\
                    text=f'Score:  {score}',font=f'Arial 17 bold')
            elif self.mode.app.splashScreenMode.playerChooseIndex ==\
                self.mode.app.splashScreenMode.player2Index:
                coins = self.mode.player2.coins
                attackValue = self.mode.player2.attackValue
                levels = self.mode.levels
                score = self.mode.score
                skill1AttackValue = self.mode.skill1.attackValue
                skill2AttackValue = self.mode.skill2.attackValue
                canvas.create_image(self.panelX,self.panelY,\
                            image=ImageTk.PhotoImage(self.panelImage))
                canvas.create_text(self.panelX-130,self.panelY-100,\
                    fill = 'black',text=f'Coins:  {coins}',\
                    font=f'Arial 17 bold', anchor='nw')
                canvas.create_text(self.panelX-130,self.panelY-60,\
                    fill = 'black', anchor='nw',\
                    text=f'AttackValue:  {attackValue}',font=f'Arial 17 bold')
                canvas.create_text(self.panelX-130,self.panelY-20,\
                    fill = 'black', anchor='nw',\
                    text=f'Skill1AttackValue:  {skill1AttackValue}',\
                    font=f'Arial 17 bold')
                canvas.create_text(self.panelX-130,self.panelY+20,\
                    fill = 'black', anchor='nw',\
                    text=f'Skill2AttackValue:  {skill2AttackValue}',\
                    font=f'Arial 17 bold')
                canvas.create_text(self.panelX-130,self.panelY+60,\
                    fill = 'black', anchor='nw',\
                    text=f'Levels:  {levels}',font=f'Arial 17 bold')
                canvas.create_text(self.panelX-130,self.panelY+100,\
                    fill = 'black', anchor='nw',\
                    text=f'Score:  {score}',font=f'Arial 17 bold')

        

class playerChoose(object):
    def __init__(self,mode):
        self.mode = mode
        self.chooseSprites = []
        for i in range(1,50):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'frame-{i}.gif')
            image = self.mode.scaleImage(image,3.3)
            self.chooseSprites.append(image)
            #(width, height) = image.size
            #self.skill1Sizes[i] = (width, height)
        #image from http://www.aigei.com/
        player1Image = self.mode.loadImage('playerChoose1.png')
        #image from http://www.aigei.com/
        player2Image = self.mode.loadImage('playerChoose2.png')
        #image from http://www.aigei.com/
        playerUnder = self.mode.loadImage('playerUnder.png')
        #image created by myself
        chooseText = self.mode.loadImage('chooseText.png')
        #image from http://www.aigei.com/
        start = self.mode.loadImage('start.png')
        self.player1Image = self.mode.scaleImage(player1Image,1.2)
        self.player2Image = self.mode.scaleImage(player2Image,1.2)
        self.playerUnder = self.mode.scaleImage(playerUnder,1.2)
        self.chooseText = self.mode.scaleImage(chooseText,2)
        self.start = self.mode.scaleImage(start,1.2)
        self.player1Width, self.player1Height = self.player1Image.size
        self.player2Width, self.player2Height = self.player2Image.size
        self.startWidth, self.startHeight = self.start.size
        self.player1X, self.player1Y =\
            self.mode.width//4.7,self.mode.height//2
        self.player2X, self.player2Y =\
            self.mode.width//2.7,self.mode.height//2
        self.startX, self.startY =\
            self.mode.width//2, self.mode.height//10*9
        
    def draw(self,canvas):
        sprite = self.chooseSprites[self.mode.spriteCounterChoose]
        canvas.create_image(self.mode.width//2,self.mode.height//2,\
                            image=ImageTk.PhotoImage(sprite))
        if self.mode.app.splashScreenMode.playerChooseIndex ==\
            self.mode.app.splashScreenMode.player1Index:
            canvas.create_image(self.player1X,\
                                self.player1Y+self.player1Height//2-10,\
                                image=ImageTk.PhotoImage(self.playerUnder))
        if self.mode.app.splashScreenMode.playerChooseIndex ==\
            self.mode.app.splashScreenMode.player2Index:
            canvas.create_image(self.player2X,\
                                self.player2Y+self.player2Height//2-10,\
                                image=ImageTk.PhotoImage(self.playerUnder))
        canvas.create_image(self.player1X, self.player1Y,\
                            image=ImageTk.PhotoImage(self.player1Image))
        canvas.create_image(self.player2X, self.player2Y,\
                            image=ImageTk.PhotoImage(self.player2Image))
        canvas.create_image(self.mode.width//2, self.mode.height//10,\
                            image=ImageTk.PhotoImage(self.chooseText))
        canvas.create_image(self.startX, self.startY,\
                            image=ImageTk.PhotoImage(self.start))

class gameMap(object):
    def __init__(self,mode):
        self.mode = mode
        #image from https://dnf.qq.com/
        self.tempMap = self.mode.loadImage('bingxin2.jpg').\
            transpose(Image.FLIP_LEFT_RIGHT) 
        self.width, self.height = self.tempMap.size
        ratio = mode.width/self.width
        self.map = self.mode.scaleImage(self.tempMap,ratio) 
    
    def draw(self,canvas):
        canvas.create_image(self.mode.width//2,self.mode.height//2,\
                            image=ImageTk.PhotoImage(self.map))

class gameMap2(object):
    def __init__(self,mode):
        self.mode = mode
        #image from https://dnf.qq.com/
        self.tempMap = self.mode.loadImage('xuankong3.jpg')
        self.width, self.height = self.tempMap.size
        ratio = mode.width/self.width
        self.map = self.mode.scaleImage(self.tempMap,ratio) 
    
    def draw(self,canvas):
        canvas.create_image(self.mode.width//2,self.mode.height//2,\
                            image=ImageTk.PhotoImage(self.map))


class coin(object):
    def __init__(self,mode,x,y,value):
        self.mode = mode
        self.x = x
        self.y = y
        self.coinValue = value #the value of the coin
        #image from https://dnf.qq.com/
        self.coinImage = self.mode.loadImage('coin2.png')
        self.width, self.height = self.coinImage.size
        ratio = mode.width/self.width
        self.coinImage = self.mode.scaleImage(self.coinImage,0.4)
    
    def draw(self,canvas):
        canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(self.coinImage))

class base(object):
    def __init__(self,mode):
        self.mode = mode
        self.baseSprites = []
        self.baseAttackSprites = []
        self.attackMove()
        self.initPosition()
        for i in range(1,13):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'base{i}.png')
            self.baseSprites.append(image)
        self.width, self.height = self.baseSprites[0].size
        self.attackValue = 100
        self.attackDistance = 500
        self.hpImage()
        self.hp = 70000
        self.fullHp = 70000
    
    def initPosition(self):
        self.x = 70
        self.y = 260
    
    def attackMove(self):
        for i in range(1,7):
            #image from http://www.aigei.com/
            image = self.mode.loadImage(f'baseattack{i}.png')
            image = self.mode.scaleImage(image,0.3)
            self.baseAttackSprites.append(image)
        #(width, height) = image.size
        #self.attackSizes = height
    
    def hpImage(self):
        self.hps = []
        self.hpsSizes = dict()
        for i in range(0,5):
            #image from https://dnf.qq.com/
            image = self.mode.loadImage(f'playerFull{i}.png')
            image = self.mode.scaleImage(image,0.4)
            self.hps.append(image)
            (width, height) = image.size
            # get size of every getAttack
            self.hpsSizes[i] = (width, height)
            
    def draw(self,canvas):
        sprite = self.baseSprites[self.mode.spriteCounterBase]
        attackSprite = self.baseAttackSprites\
                       [self.mode.spriteCounterBaseAttack]
        canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(sprite))
        if self.mode.player1.status == self.mode.player1.dead or\
            self.mode.player2.status == self.mode.player2.dead:
            for enemy in self.mode.enemySet:
                if enemy.getDistanceToBase() <= self.attackDistance:
                    canvas.create_image(enemy.x,enemy.y,\
                                image=ImageTk.PhotoImage(attackSprite))
        #draw hp
        for i in range(5):
            if (1-(i+1)*0.2)*self.fullHp < self.hp <= (1-i*0.2)*self.fullHp:
                canvas.create_image\
                (self.x,self.y-self.height//2-30,\
                image=ImageTk.PhotoImage(self.hps[i]))
                break

class base2(base):
    def initPosition(self):
        self.x = 90
        self.y = 360

    def draw(self,canvas):
        sprite = self.baseSprites[self.mode.spriteCounterBase]
        
        canvas.create_image(self.x,self.y,\
                            image=ImageTk.PhotoImage(sprite))
        
        #draw hp
        for i in range(5):
            if (1-(i+1)*0.2)*self.fullHp < self.hp <= (1-i*0.2)*self.fullHp:
                canvas.create_image\
                (self.x,self.y-self.height//2-30,\
                image=ImageTk.PhotoImage(self.hps[i]))
                break
    

def runCreativeSidescroller():
    KnightsTemplar(width=1000, height=500)

def main():
    runCreativeSidescroller()
    
if __name__ == '__main__':
    main()
```
