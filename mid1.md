# 1번 답안
## 1번 문항 : 무지개 블록깨기 게임을 C언어로 작성하시오. (그래픽 라이브러리 활용)

```
나의 프롬프트 :
무지개 블록 깨기 게임을 만들려고해.
환경은 C언어로 Visual Studio에서 raylib를 활용하여 할거야.
단, raylib는 최신 버전 nuget으로 설치를 하고, 코드는 너무 길게는 만들지마.
실행 되도록 확인해서 코드 만들어줘.
```

#include "raylib.h"
#include <math.h>
#include <stdbool.h>
#include <stdio.h>

#define SCREEN_WIDTH  1000
#define SCREEN_HEIGHT 720

#define BRICK_COLS 10
#define BRICK_ROWS 7
#define BRICK_W    84.0f
#define BRICK_H    25.0f
#define BRICK_GAP  8.0f
#define BRICK_TOP  125.0f

#define PADDLE_W   120.0f
#define PADDLE_H   16.0f
#define BALL_R     9.0f

#define MAX_LIVES  3

typedef enum GameState {
    GAME_PLAYING,
    GAME_PAUSED,
    GAME_OVER,
    GAME_WIN
} GameState;

typedef struct Brick {
    Rectangle rect;
    Color color;
    bool alive;
} Brick;

typedef struct Ball {
    Vector2 pos;
    Vector2 vel;
    float radius;
} Ball;

static const Color rainbow[7] = {
    {255,  65,  85, 255},  // Red
    {255, 145,  55, 255},  // Orange
    {255, 220,  65, 255},  // Yellow
    { 65, 220, 125, 255},  // Green
    { 65, 175, 255, 255},  // Blue
    {100, 105, 255, 255},  // Indigo
    {205, 105, 255, 255}   // Violet
};

static Brick bricks[BRICK_ROWS][BRICK_COLS];

static Ball ball;
static Rectangle paddle;

static int score = 0;
static int lives = MAX_LIVES;
static int remainingBricks = BRICK_ROWS * BRICK_COLS;
static GameState gameState = GAME_PLAYING;

static float ballSpeed = 390.0f;

// --------------------------------------------------
// Utility
// --------------------------------------------------

static float ClampFloat(float value, float minValue, float maxValue)
{
    if (value < minValue) return minValue;
    if (value > maxValue) return maxValue;
    return value;
}

static void DrawCenteredText(const char* text, int y,
    int fontSize, Color color)
{
    int width = MeasureText(text, fontSize);
    DrawText(text, (SCREEN_WIDTH - width) / 2,
        y, fontSize, color);
}

static void InitBricks(void)
{
    float totalWidth =
        BRICK_COLS * BRICK_W + (BRICK_COLS - 1) * BRICK_GAP;

    float startX = (SCREEN_WIDTH - totalWidth) / 2.0f;

    remainingBricks = 0;

    for (int row = 0; row < BRICK_ROWS; row++)
    {
        for (int col = 0; col < BRICK_COLS; col++)
        {
            Brick* brick = &bricks[row][col];

            brick->rect = (Rectangle){
                startX + col * (BRICK_W + BRICK_GAP),
                BRICK_TOP + row * (BRICK_H + BRICK_GAP),
                BRICK_W,
                BRICK_H
            };

            // 각 행이 무지개색 순서로 배치됩니다.
            brick->color = rainbow[row % 7];
            brick->alive = true;

            remainingBricks++;
        }
    }
}

static void ResetBallAndPaddle(void)
{
    paddle = (Rectangle){
        SCREEN_WIDTH / 2.0f - PADDLE_W / 2.0f,
        SCREEN_HEIGHT - 60.0f,
        PADDLE_W,
        PADDLE_H
    };

    ball.radius = BALL_R;
    ball.pos = (Vector2){
        SCREEN_WIDTH / 2.0f,
        SCREEN_HEIGHT - 90.0f
    };

    // 처음부터 오른쪽 위 방향으로 출발합니다.
    ball.vel = (Vector2){
        ballSpeed * 0.55f,
        -ballSpeed * 0.84f
    };
}

static void ResetGame(void)
{
    score = 0;
    lives = MAX_LIVES;
    ballSpeed = 390.0f;
    gameState = GAME_PLAYING;

    InitBricks();
    ResetBallAndPaddle();
}

// 원과 사각형의 충돌 검사
static bool CircleRectCollision(Vector2 center, float radius,
    Rectangle rect)
{
    float closestX = ClampFloat(
        center.x, rect.x, rect.x + rect.width);

    float closestY = ClampFloat(
        center.y, rect.y, rect.y + rect.height);

    float dx = center.x - closestX;
    float dy = center.y - closestY;

    return dx * dx + dy * dy <= radius * radius;
}

// --------------------------------------------------
// Update
// --------------------------------------------------

static void UpdatePaddle(void)
{
    float dt = GetFrameTime();
    float paddleSpeed = 620.0f;

    if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A))
        paddle.x -= paddleSpeed * dt;

    if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D))
        paddle.x += paddleSpeed * dt;

    // 마우스를 움직여도 패들을 조작할 수 있습니다.
    if (IsMouseButtonDown(MOUSE_BUTTON_LEFT))
    {
        paddle.x = GetMouseX() - paddle.width / 2.0f;
    }

    paddle.x = ClampFloat(
        paddle.x, 0.0f, SCREEN_WIDTH - paddle.width);
}

static void UpdateBall(void)
{
    float dt = GetFrameTime();

    // 프레임 간격이 길어지는 상황에서 이동량을 제한합니다.
    if (dt > 0.033f) dt = 0.033f;

    Vector2 previous = ball.pos;

    ball.pos.x += ball.vel.x * dt;
    ball.pos.y += ball.vel.y * dt;

    // 왼쪽 / 오른쪽 벽
    if (ball.pos.x - ball.radius <= 0.0f)
    {
        ball.pos.x = ball.radius;
        ball.vel.x = fabsf(ball.vel.x);
    }
    else if (ball.pos.x + ball.radius >= SCREEN_WIDTH)
    {
        ball.pos.x = SCREEN_WIDTH - ball.radius;
        ball.vel.x = -fabsf(ball.vel.x);
    }

    // 천장
    if (ball.pos.y - ball.radius <= 78.0f)
    {
        ball.pos.y = 78.0f + ball.radius;
        ball.vel.y = fabsf(ball.vel.y);
    }

    // 패들 충돌: 공이 아래로 이동할 때만 반사
    if (ball.vel.y > 0.0f &&
        CircleRectCollision(ball.pos, ball.radius, paddle))
    {
        ball.pos.y = paddle.y - ball.radius;

        // 패들 중앙에서 맞았는지, 가장자리에서 맞았는지에 따라
        // 공의 반사 방향이 달라집니다.
        float paddleCenter = paddle.x + paddle.width / 2.0f;
        float hit = (ball.pos.x - paddleCenter)
            / (paddle.width / 2.0f);

        hit = ClampFloat(hit, -0.92f, 0.92f);

        float speed = sqrtf(
            ball.vel.x * ball.vel.x +
            ball.vel.y * ball.vel.y);

        float angle = hit * 1.05f;

        ball.vel.x = speed * sinf(angle);
        ball.vel.y = -speed * cosf(angle);

        // 공이 너무 수평으로 움직이지 않도록 합니다.
        if (fabsf(ball.vel.y) < speed * 0.30f)
        {
            ball.vel.y = -speed * 0.30f;
            float length = sqrtf(
                ball.vel.x * ball.vel.x +
                ball.vel.y * ball.vel.y);

            ball.vel.x *= speed / length;
            ball.vel.y *= speed / length;
        }
    }

    // 블록 충돌
    for (int row = 0; row < BRICK_ROWS; row++)
    {
        for (int col = 0; col < BRICK_COLS; col++)
        {
            Brick* brick = &bricks[row][col];

            if (!brick->alive)
                continue;

            if (CircleRectCollision(
                ball.pos, ball.radius, brick->rect))
            {
                brick->alive = false;
                remainingBricks--;
                score += 10;

                // 충돌 직전의 위치로 어느 방향에서
                // 블록에 닿았는지 판정합니다.
                if (previous.y + ball.radius <= brick->rect.y)
                {
                    ball.vel.y = -fabsf(ball.vel.y);
                }
                else if (previous.y - ball.radius >=
                    brick->rect.y + brick->rect.height)
                {
                    ball.vel.y = fabsf(ball.vel.y);
                }
                else
                {
                    ball.vel.x = -ball.vel.x;
                }

                // 블록 하나당 공의 반사는 한 번만 처리합니다.
                goto brick_collision_done;
            }
        }
    }

brick_collision_done:

    // 바닥에 떨어지면 목숨 차감
    if (ball.pos.y - ball.radius > SCREEN_HEIGHT)
    {
        lives--;

        if (lives <= 0)
        {
            gameState = GAME_OVER;
        }
        else
        {
            ResetBallAndPaddle();
        }
    }

    // 마지막 블록까지 파괴하면 승리
    if (remainingBricks <= 0)
    {
        gameState = GAME_WIN;
    }
}

// --------------------------------------------------
// Draw
// --------------------------------------------------

static void DrawBackground(void)
{
    ClearBackground((Color) { 12, 16, 36, 255 });

    // 장식용 별
    for (int i = 0; i < 48; i++)
    {
        int x = (i * 197 + 53) % SCREEN_WIDTH;
        int y = (i * 113 + 29) % SCREEN_HEIGHT;

        DrawCircle(x, y, (i % 3 == 0) ? 2.0f : 1.0f,
            (Color) {
            100, 125, 190, 100
        });
    }

    // 상단 HUD 영역
    DrawRectangle(0, 0, SCREEN_WIDTH, 76,
        (Color) {
        20, 25, 55, 255
    });

    DrawLine(0, 76, SCREEN_WIDTH, 76,
        (Color) {
        85, 100, 180, 180
    });

    DrawText("RAINBOW", 22, 16, 25, RAYWHITE);
    DrawText("BREAKOUT", 22, 42, 14,
        (Color) {
        150, 170, 255, 255
    });

    DrawText("SCORE", 390, 14, 12, LIGHTGRAY);

    char scoreText[32];
    snprintf(scoreText, sizeof(scoreText), "%06d", score);
    DrawText(scoreText, 390, 32, 25, GOLD);

    DrawText("LIVES", 745, 14, 12, LIGHTGRAY);

    for (int i = 0; i < MAX_LIVES; i++)
    {
        Color lifeColor = (i < lives) ? RED :
            (Color) {
            65, 70, 95, 255
        };

        DrawCircle(755 + i * 26, 49, 8, lifeColor);
    }

    char blocksText[40];
    snprintf(blocksText, sizeof(blocksText),
        "BLOCKS: %d", remainingBricks);

    DrawText(blocksText, 855, 32, 17,
        (Color) {
        180, 205, 255, 255
    });
}

static void DrawBricks(void)
{
    for (int row = 0; row < BRICK_ROWS; row++)
    {
        for (int col = 0; col < BRICK_COLS; col++)
        {
            Brick* brick = &bricks[row][col];

            if (!brick->alive)
                continue;

            // 은은한 그림자
            Rectangle shadow = brick->rect;
            shadow.y += 4.0f;

            DrawRectangleRounded(
                shadow, 0.28f, 6,
                (Color) {
                0, 0, 0, 90
            });

            DrawRectangleRounded(
                brick->rect, 0.28f, 6, brick->color);

            // 블록 위쪽의 빛 반사 효과
            DrawRectangleRounded(
                (Rectangle) {
                brick->rect.x + 4,
                    brick->rect.y + 3,
                    brick->rect.width - 8,
                    4
            },
                0.5f, 4,
                (Color) {
                255, 255, 255, 100
            });
        }
    }
}

static void DrawPaddle(void)
{
    DrawRectangleRounded(
        (Rectangle) {
        paddle.x - 3,
            paddle.y + 4,
            paddle.width + 6,
            paddle.height
    },
        0.6f, 8,
        (Color) {
        0, 0, 0, 100
    });

    DrawRectangleRounded(
        paddle, 0.6f, 8,
        (Color) {
        100, 190, 255, 255
    });

    DrawRectangleRounded(
        (Rectangle) {
        paddle.x + 10,
            paddle.y + 3,
            paddle.width - 20,
            4
    },
        0.5f, 4, RAYWHITE);
}

static void DrawBall(void)
{
    // 바깥쪽 광채
    DrawCircleV(ball.pos, ball.radius + 6,
        (Color) {
        100, 175, 255, 35
    });

    DrawCircleV(ball.pos, ball.radius + 3,
        (Color) {
        130, 205, 255, 90
    });

    DrawCircleV(ball.pos, ball.radius, RAYWHITE);

    DrawCircle(
        (int)(ball.pos.x - 2),
        (int)(ball.pos.y - 3),
        2.5f, SKYBLUE);
}

static void DrawOverlay(const char* title,
    const char* subtitle,
    Color titleColor)
{
    DrawRectangle(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT,
        (Color) {
        5, 8, 25, 190
    });

    DrawCenteredText(title, 275, 48, titleColor);
    DrawCenteredText(subtitle, 345, 21, RAYWHITE);
    DrawCenteredText("Press R to restart", 410, 18,
        (Color) {
        180, 195, 240, 255
    });
}

static void DrawGame(void)
{
    DrawBackground();
    DrawBricks();
    DrawPaddle();
    DrawBall();

    DrawText("Move: A/D or arrows | Mouse: hold left button",
        20, SCREEN_HEIGHT - 25, 14,
        (Color) {
        145, 160, 200, 255
    });

    DrawText("SPACE: Pause", SCREEN_WIDTH - 155,
        SCREEN_HEIGHT - 25, 14,
        (Color) {
        145, 160, 200, 255
    });

    if (gameState == GAME_PAUSED)
    {
        DrawOverlay("PAUSED", "Press SPACE to continue",
            (Color) {
            255, 225, 95, 255
        });
    }
    else if (gameState == GAME_OVER)
    {
        DrawOverlay("GAME OVER", "You ran out of lives!",
            (Color) {
            255, 95, 120, 255
        });
    }
    else if (gameState == GAME_WIN)
    {
        DrawOverlay("YOU WIN!", "Every rainbow brick is gone!",
            (Color) {
            120, 255, 175, 255
        });
    }
}

// --------------------------------------------------
// Main
// --------------------------------------------------

int main(void)
{
    SetConfigFlags(FLAG_VSYNC_HINT);
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT,
        "Rainbow Breakout - raylib");
    SetTargetFPS(60);

    ResetGame();

    while (!WindowShouldClose())
    {
        if (IsKeyPressed(KEY_R))
        {
            ResetGame();
        }

        if (IsKeyPressed(KEY_SPACE))
        {
            if (gameState == GAME_PLAYING)
                gameState = GAME_PAUSED;
            else if (gameState == GAME_PAUSED)
                gameState = GAME_PLAYING;
        }

        if (gameState == GAME_PLAYING)
        {
            UpdatePaddle();
            UpdateBall();
        }

        BeginDrawing();
        DrawGame();
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
