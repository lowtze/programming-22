---
path: '/part-13/3-events'
title: 'Events'
hidden: false
---

<text-box variant='learningObjectives' name="Learning objectives">

After this section

- You will be familiar with Pygame events
- You will be able to write a program which reacts to key presses
- You will be able to write a program which reacts to mouse events

</text-box>

Thus far our main loops have only executed predetermined animations and reacted to only `pygame.QUIT` type events, even though the loop gets a ist of all events from the operating system. Let's get to grips with some other types of events.

## Handling events

Seuraava koodi näyttää, mitä tapahtumia syntyy ohjelman suorituksen aikana:

```python
import pygame

pygame.init()
naytto = pygame.display.set_mode((640, 480))

while True:
    for tapahtuma in pygame.event.get():
        print(tapahtuma)
        if tapahtuma.type == pygame.QUIT:
            exit()
```

Kun ohjelmaa käytetään hetki, se voi tulostaa esimerkiksi seuraavanlaisia tapahtumia:

```x
<Event(4-MouseMotion {'pos': (495, 274), 'rel': (495, 274), 'buttons': (0, 0, 0), 'window': None})>
<Event(4-MouseMotion {'pos': (494, 274), 'rel': (-1, 0), 'buttons': (0, 0, 0), 'window': None})>
<Event(4-MouseMotion {'pos': (492, 274), 'rel': (-2, 0), 'buttons': (0, 0, 0), 'window': None})>
<Event(4-MouseMotion {'pos': (491, 274), 'rel': (-1, 0), 'buttons': (0, 0, 0), 'window': None})>
<Event(5-MouseButtonDown {'pos': (491, 274), 'button': 1, 'window': None})>
<Event(6-MouseButtonUp {'pos': (491, 274), 'button': 1, 'window': None})>
<Event(2-KeyDown {'unicode': 'a', 'key': 97, 'mod': 0, 'scancode': 38, 'window': None})>
<Event(3-KeyUp {'key': 97, 'mod': 0, 'scancode': 38, 'window': None})>
<Event(2-KeyDown {'unicode': 'b', 'key': 98, 'mod': 0, 'scancode': 56, 'window': None})>
<Event(3-KeyUp {'key': 98, 'mod': 0, 'scancode': 56, 'window': None})>
<Event(2-KeyDown {'unicode': 'c', 'key': 99, 'mod': 0, 'scancode': 54, 'window': None})>
<Event(3-KeyUp {'key': 99, 'mod': 0, 'scancode': 54, 'window': None})>
<Event(12-Quit {})>
```

Tässä ensimmäiset tapahtumat liittyvät hiiren käyttämiseen, seuraavat tapahtumat näppäimistön käyttämiseen ja viimeinen tapahtuma sulkee ohjelman. Jokaisella tapahtumalla on tyyppi ja mahdollisesti lisätietoa, josta voi päätellä esimerkiksi hiiren sijainnin tai painetun näppäimen.

Tapahtumia voi etsiä Pygamen dokumentaatiosta mutta usein tehokas tapa löytää sopiva tapahtuma on käyttää yllä olevaa koodia ja tutkia, millainen tapahtuma syntyy, kun ohjelmassa tapahtuu haluttu asia.

## Näppäimistön käsittely

Seuraava ohjelma tunnistaa tapahtumat, joissa käyttäjä painaa oikealle tai vasemmalle nuolinäppäintä. Ohjelma tulostaa testiksi tiedon näppäimen painamisesta.

```python
import pygame

pygame.init()
naytto = pygame.display.set_mode((640, 480))

while True:
    for tapahtuma in pygame.event.get():
        if tapahtuma.type == pygame.KEYDOWN:
            if tapahtuma.key == pygame.K_LEFT:
                print("vasemmalle")
            if tapahtuma.key == pygame.K_RIGHT:
                print("oikealle")

        if tapahtuma.type == pygame.QUIT:
            exit()
```

Tässä vakiot `pygame.K_LEFT` ja `pygame.K_RIGHT` tarkoittavat nuolinäppäimiä vasemmalle ja oikealle. Näppäimistön eri näppäimiä vastaavat vakiot on listattu [Pygamen dokumentaatiossa](https://www.pygame.org/docs/ref/key.html#key-constants-label).

Esimerkiksi kun käyttäjä painaa ensin kahdesti oikealle, sitten kerran vasemmalle ja lopuksi kerran oikealle, ohjelman tulostus on seuraava:

```x
oikealle
oikealle
vasemmalle
oikealle
```

Voimme nyt tehdä ohjelman, jossa käyttäjä pystyy liikuttamaan hahmoa oikealle ja vasemmalle nuolinäppäimillä. Tämä onnistuu seuraavasti:

```python
import pygame

pygame.init()
naytto = pygame.display.set_mode((640, 480))

robo = pygame.image.load("robot.png")
x = 0
y = 480-robo.get_height()

while True:
    for tapahtuma in pygame.event.get():
        if tapahtuma.type == pygame.KEYDOWN:
            if tapahtuma.key == pygame.K_LEFT:
                x -= 10
            if tapahtuma.key == pygame.K_RIGHT:
                x += 10

        if tapahtuma.type == pygame.QUIT:
            exit()

    naytto.fill((0, 0, 0))
    naytto.blit(robo, (x, y))
    pygame.display.flip()
```

Ohjelman suoritus voi näyttää seuraavalta:

<img src="pygame_move_robot.gif">

Tässä muuttujat `x` ja `y` sisältävät hahmon sijainnin. Käyttäjä pystyy muuttamaan muuttujaa `x`, ja muuttuja `y` on asetettu niin, että hahmo on ikkunan alalaidassa. Kun käyttäjä painaa vasemmalle tai oikealle nuolinäppäintä, hahmo liikkuu vastaavasti 10 pikseliä oikealle tai vasemmalle.

Yllä oleva ohjelma toimii muuten hyvin, mutta pelikokemuksessa on puutteena, että näppäintä pitää painaa uudestaan aina, kun haluaa liikkua askeleen oikealle tai vasemmalle. Olisi parempi, että voi pitää näppäintä pohjassa ja hahmo liikkuu niin kauan, kuin näppäin on pohjassa. Seuraava koodi mahdollistaa tämän:

```python
import pygame

pygame.init()
naytto = pygame.display.set_mode((640, 480))

robo = pygame.image.load("robot.png")
x = 0
y = 480-robo.get_height()

oikealle = False
vasemmalle = False

kello = pygame.time.Clock()

while True:
    for tapahtuma in pygame.event.get():
        if tapahtuma.type == pygame.KEYDOWN:
            if tapahtuma.key == pygame.K_LEFT:
                vasemmalle = True
            if tapahtuma.key == pygame.K_RIGHT:
                oikealle = True

        if tapahtuma.type == pygame.KEYUP:
            if tapahtuma.key == pygame.K_LEFT:
                vasemmalle = False
            if tapahtuma.key == pygame.K_RIGHT:
                oikealle = False

        if tapahtuma.type == pygame.QUIT:
            exit()

    if oikealle:
        x += 2
    if vasemmalle:
        x -= 2

    naytto.fill((0, 0, 0))
    naytto.blit(robo, (x, y))
    pygame.display.flip()

    kello.tick(60)
```

Koodissa on nyt muuttujat `oikealle` ja `vasemmalle`, joissa pidetään tietoa siitä, kuuluuko hahmon liikkua tällä hetkellä oikealle tai vasemmalle. Kun käyttäjä painaa alas nuolinäppäimen, vastaava muuttuja saa arvon `True`, ja kun käyttäjä nostaa alas nuolinäppäimen, vastaava muuttuja saa arvon `False`.

Hahmon liike on tahdistettu kellon avulla niin, että liikkumista tapahtuu 60 kertaa sekunnissa. Jos nuolinäppäin on alhaalla, hahmo liikkuu 2 pikseliä oikealle tai vasemmalle. Tämän seurauksena hahmo liikkuu 120 pikseliä sekunnissa, jos nuolinäppäin on painettuna.

<programming-exercise name='Four directions' tmcname='part13-11_four_directions'>

Please write a program where the player can move a robot in four directions with the arrow keys on the keyboard. The end result should look like this:

<img src="pygame_four_directions.gif">

</programming-exercise>

<programming-exercise name='Four walls' tmcname='part13-12_four_walls'>

Please improve the program in the previous exercise so that the robot cannot pass outside the window in any of the four directions. The end result should look like this:

<img src="pygame_four_walls.gif">

</programming-exercise>

<programming-exercise name='Two players' tmcname='part13-13_two_players'>

Please write a program where two players each direct their own robot. One of the players should use the arrow keys while the other could use, for example, the w-s-a-d keys. The end result should look like this:

<img src="pygame_two_players.gif">

</programming-exercise>

## Hiiren käsittely

Seuraava koodi tunnistaa tapahtumat, jossa käyttäjä painaa hiiren nappia ikkunan alueella:

```python
import pygame

pygame.init()
naytto = pygame.display.set_mode((640, 480))

while True:
    for tapahtuma in pygame.event.get():
        if tapahtuma.type == pygame.MOUSEBUTTONDOWN:
            print("painoit nappia", tapahtuma.button, "kohdassa", tapahtuma.pos)

        if tapahtuma.type == pygame.QUIT:
            exit()
```

Ohjelman suoritus voi näyttää tältä:

```x
painoit nappia 1 kohdassa (82, 135)
painoit nappia 1 kohdassa (369, 135)
painoit nappia 1 kohdassa (269, 297)
painoit nappia 3 kohdassa (515, 324)
```

Tässä nappi 1 tarkoittaa hiiren vasenta nappia ja nappi 3 tarkoittaa hiiren oikeaa nappia.

Seuraava ohjelma yhdistää hiiren käsittelyn ja kuvan piirtämisen. Kun käyttäjä painaa hiirellä ikkunan alueella, robotti piirretään hiiren kohtaan.

```python
import pygame

pygame.init()
naytto = pygame.display.set_mode((640, 480))

robo = pygame.image.load("robot.png")

while True:
    for tapahtuma in pygame.event.get():
        if tapahtuma.type == pygame.MOUSEBUTTONDOWN:
            x = tapahtuma.pos[0]-robo.get_width()/2
            y = tapahtuma.pos[1]-robo.get_height()/2

            naytto.fill((0, 0, 0))
            naytto.blit(robo, (x, y))
            pygame.display.flip()

        if tapahtuma.type == pygame.QUIT:
            exit()
```

Ohjelman suoritus voi näyttää tältä:

<img src="pygame_cursor.gif">

Seuraava ohjelma puolestaan toteuttaa animaation, jossa robotti seuraa hiirtä. Robotin sijainti on muuttujissa `robo_x` ja `robo_y`, ja kun hiiri liikkuu, sen sijainti merkitään muuttujiin `kohde_x` ja `kohde_y`. Jos robotti ei ole hiiren kohdalla, se liikkuu sopivaan suuntaan.

```python
import pygame

pygame.init()
naytto = pygame.display.set_mode((640, 480))

robo = pygame.image.load("robot.png")

robo_x = 0
robo_y = 0
kohde_x = 0
kohde_y = 0

kello = pygame.time.Clock()

while True:
    for tapahtuma in pygame.event.get():
        if tapahtuma.type == pygame.MOUSEMOTION:
            kohde_x = tapahtuma.pos[0]-robo.get_width()/2
            kohde_y = tapahtuma.pos[1]-robo.get_height()/2

        if tapahtuma.type == pygame.QUIT:
            exit(0)

    if robo_x > kohde_x:
        robo_x -= 1
    if robo_x < kohde_x:
        robo_x += 1
    if robo_y > kohde_y:
        robo_y -= 1
    if robo_y < kohde_y:
        robo_y += 1

    naytto.fill((0, 0, 0))
    naytto.blit(robo, (robo_x, robo_y))
    pygame.display.flip()

    kello.tick(60)
```

Ohjelman suoritus voi näyttää tältä:

<img src="pygame_cursor2.gif">

<programming-exercise name='Robot and mouse' tmcname='part13-14_robot_and_mouse'>

Please write a program where the robot follows the mouse cursor so that the centre of the robot is always directly at the mouse cursor. The end result should look like this:

<img src="pygame_robot_cursor.gif">

</programming-exercise>

<programming-exercise name='The location of the robot' tmcname='part13-15_robot_location'>

Please write a program where the robot appears at a random location within the window. When the player clicks on the robot with the mouse, the robot moves to a new location. The end result should look like this:

<img src="pygame_robot_location.gif">

</programming-exercise>