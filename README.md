# Snake-Game-C

/*
    Snake Game (Console) - C / Code::Blocks / Windows
    Features:
      - Arrow keys & WASD control
      - Growing tail, score, levels (speed up)
      - Play again prompt on game over
*/

#include <stdio.h>
#include <stdlib.h>
#include <conio.h>     // kbhit, getch
#include <windows.h>   // Sleep
#include <time.h>      // time for srand
#include <stdbool.h>

#define WIDTH   40
#define HEIGHT  20
#define MAX_TAIL (WIDTH * HEIGHT)

typedef enum { STOP = 0, LEFT, RIGHT, UP, DOWN } Direction;

typedef struct {
    int x, y;
} Point;

static bool gameOver;
static int score;
static Point head;
static Point fruit;
static Point tail[MAX_TAIL];
static int tailLen;
static Direction dir;
static int level;       // increases as you score
static int delayMs;     // decreases as level increases

// Random int in [min, max]
int rnd(int min, int max) {
    return min + rand() % (max - min + 1);
}

void placeFruit(void) {
    while (1) {
        fruit.x = rnd(1, WIDTH - 2);
        fruit.y = rnd(1, HEIGHT - 2);

        // avoid placing on snake
        bool clash = false;
        if (fruit.x == head.x && fruit.y == head.y) clash = true;
        for (int i = 0; i < tailLen && !clash; ++i) {
            if (tail[i].x == fruit.x && tail[i].y == fruit.y) clash = true;
        }
        if (!clash) break;
    }
}

void resetGame(void) {
    gameOver = false;
    score = 0;
    tailLen = 0;
    dir = STOP;
    level = 1;
    delayMs = 130;

    head.x = WIDTH / 2;
    head.y = HEIGHT / 2;

    placeFruit();
}

void draw(void) {
    // Clear screen
    // Note: system("cls") is fine for simple console games
    system("cls");

    // Top wall
    for (int i = 0; i < WIDTH; ++i) putchar('#');
    putchar('\n');

    // Playfield
    for (int y = 0; y < HEIGHT - 2; ++y) {
        for (int x = 0; x < WIDTH; ++x) {
            if (x == 0 || x == WIDTH - 1) {
                putchar('#'); // side walls
            } else {
                int drawX = x, drawY = y + 1;

                if (drawX == head.x && drawY == head.y) {
                    putchar('O'); // head
                } else if (drawX == fruit.x && drawY == fruit.y) {
                    putchar('*'); // fruit
                } else {
                    bool printed = false;
                    for (int i = 0; i < tailLen; ++i) {
                        if (tail[i].x == drawX && tail[i].y == drawY) {
                            putchar('o'); // tail
                            printed = true;
                            break;
                        }
                    }
                    if (!printed) putchar(' ');
                }
            }
        }
        putchar('\n');
    }

    // Bottom wall
    for (int i = 0; i < WIDTH; ++i) putchar('#');
    putchar('\n');

    printf("Score: %d   Level: %d   (WASD/Arrows to move, X to quit)\n", score, level);
}

void input(void) {
    if (_kbhit()) {
        int ch = _getch();

        // Arrow keys come as 224 then code
        if (ch == 224) {
            int ch2 = _getch();
            switch (ch2) {
                case 72: if (dir != DOWN)  dir = UP;    break; // Up
                case 80: if (dir != UP)    dir = DOWN;  break; // Down
                case 75: if (dir != RIGHT) dir = LEFT;  break; // Left
                case 77: if (dir != LEFT)  dir = RIGHT; break; // Right
            }
            return;
        }

        // Convert to uppercase for wasd
        if (ch >= 'a' && ch <= 'z') ch -= 32;

        switch (ch) {
            case 'W': if (dir != DOWN)  dir = UP;    break;
            case 'S': if (dir != UP)    dir = DOWN;  break;
            case 'A': if (dir != RIGHT) dir = LEFT;  break;
            case 'D': if (dir != LEFT)  dir = RIGHT; break;
            case 'X': gameOver = true;  break;
        }
    }
}

void logic(void) {
    // Move tail (from end to front)
    for (int i = tailLen - 1; i > 0; --i) {
        tail[i] = tail[i - 1];
    }
    if (tailLen > 0) {
        tail[0] = head;
    }

    // Move head
    switch (dir) {
        case LEFT:  head.x--; break;
        case RIGHT: head.x++; break;
        case UP:    head.y--; break;
        case DOWN:  head.y++; break;
        default: break; // STOP
    }

    // Collisions with wall
    if (head.x <= 0 || head.x >= WIDTH - 1 || head.y <= 0 || head.y >= HEIGHT - 1) {
        gameOver = true;
        return;
    }

    // Collisions with tail
    for (int i = 0; i < tailLen; ++i) {
        if (tail[i].x == head.x && tail[i].y == head.y) {
            gameOver = true;
            return;
        }
    }

    // Fruit eaten
    if (head.x == fruit.x && head.y == fruit.y) {
        score += 10;
        if (tailLen < MAX_TAIL) tail[tailLen++] = head;

        // Speed up every 50 points (min cap)
        if (score % 50 == 0) {
            level++;
            if (delayMs > 50) delayMs -= 10;
        }

        placeFruit();
    }
}

int playOnce(void) {
    resetGame();

    while (!gameOver) {
        draw();
        input();
        logic();
        Sleep(delayMs);
    }

    system("cls");
    printf("########################################\n");
    printf("#              GAME OVER               #\n");
    printf("########################################\n");
    printf("Final Score: %d   Level Reached: %d\n", score, level);
    printf("\nPlay again? (Y/N): ");

    int ch;
    do { ch = _getch(); } while (ch == 224); // skip arrow prefix

    if (ch >= 'a' && ch <= 'z') ch -= 32;
    return (ch == 'Y') ? 1 : 0;
}

int main(void) {
    srand((unsigned)time(NULL));

    // Hide cursor for nicer look (optional)
    HANDLE hOut = GetStdHandle(STD_OUTPUT_HANDLE);
    CONSOLE_CURSOR_INFO cci;
    if (GetConsoleCursorInfo(hOut, &cci)) {
        cci.bVisible = FALSE;
        SetConsoleCursorInfo(hOut, &cci);
    }

    do {
        // Title screen
        system("cls");
        printf("########################################\n");
        printf("#              S N A K E               #\n");
        printf("########################################\n");
        printf("#  Controls:                           #\n");
        printf("#   - Arrow Keys or W A S D            #\n");
        printf("#   - X to quit                        #\n");
        printf("########################################\n");
        printf("Press any key to start...");
        _getch();
    } while (playOnce());

    // Restore cursor
    if (GetConsoleCursorInfo(hOut, &cci)) {
        cci.bVisible = TRUE;
        SetConsoleCursorInfo(hOut, &cci);
    }

    return 0;
}

