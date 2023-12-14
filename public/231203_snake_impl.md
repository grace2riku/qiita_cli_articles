---
title: Snakeゲームを移植した話
tags:
  - Spresense
private: false
updated_at: '2023-12-04T23:25:24+09:00'
id: 13bdfc7c649f42f8e489
organization_url_name: rymansat
slide: false
ignorePublish: false
---
この記事は[Spresense Advent Calendar 2023](https://qiita.com/advent-calendar/2023/spresense)の3日目（12/3分）です。

# 概要
snekeゲームをSpresenseに移植してみた記事です。

<blockquote class="twitter-tweet" data-media-max-width="560"><p lang="ja" dir="ltr">Spresenseでsnakeゲームしてみました。<br><br>これから投稿するQiita Advent Calender 2023 【Spresense】の12/3分の記事のテーマです。 <a href="https://t.co/V2Coq1Hedq">pic.twitter.com/V2Coq1Hedq</a></p>&mdash; k-abe@組み込まれた猫使い🧙‍♂️ (@juraruming) <a href="https://twitter.com/juraruming/status/1731679534133436634?ref_src=twsrc%5Etfw">December 4, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# コード
コードはGitHubリポジトリに置きました。

https://github.com/grace2riku/spresense_snake

# 確認環境
## ハードウェア
つぎのハードウェアで確認しました。

* Spresenseメインボード
* Spresense拡張ボード
* LCD ILI9341

## ソフトウェア
Spresense SDKで確認しました。

* [spresense SDK v3.0.0 (2023/03/13)](https://developer.sony.com/develop/spresense/docs/release_sdk_ja.html)

# 確認手順
つぎの手順で確認します。

## 環境構築
### Spresense SDK 開発ツールのセットアップ
事前にSpresense SDKでの開発 -> Spresense SDK スタートガイド (CLI 版) -> [2. 開発ツールのセットアップ](https://developer.sony.com/spresense/development-guides/sdk_set_up_ja.html#_%E9%96%8B%E7%99%BA%E3%83%84%E3%83%BC%E3%83%AB%E3%81%AE%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97)を実施し、開発環境を構築しておきます。

### ユーザアプリの追加
Spresense SDKでの開発 -> Spresense SDK スタートガイド (CLI 版) -> 6. ユーザーアプリの追加方法 -> [6.3. ツールを使用する](https://developer.sony.com/spresense/development-guides/sdk_set_up_ja.html#_%E3%83%84%E3%83%BC%E3%83%AB%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%99%E3%82%8B)を参照し、ユーザアプリを追加します。

今回はつぎのコマンドでユーザアプリを追加しました。

```
tools/mkappsdir.py snake “snake_command”
tools/mkcmd.py -d snake snake “snake_command”
```

## コンフィグレーション
spresense/sdkディレクトリで次のコマンドを実行します。

```
tools/config.py examples/pdcurses
```

Macの場合、次のディレクトリでした。
```
/Users/ユーザ名/spresense/sdk
```

## ソースコード準備
snakeゲームはOSSのソースコードを流用させてもらいます。
流用させてもらったsnakeゲームのGitHubリポジトリ

https://github.com/Sheep42/ncurses-snake

ライセンスはMITです。

snake.cのコードをsnake/snake_main.cにコピーします。

## コード変更
snake/snake_main.cをつぎのように変更します。
今回はシリアルコンソールからのキー入力でスネークが動く動作にします。

```diff_c
#include <time.h>
-#include <ncurses.h>
+#include "graphics/curses.h"
#include <unistd.h>
#include <stdlib.h>

+#include "debug.h"
+#include <fcntl.h>

#define DELAY 30000
#define TIMEOUT 10 

/* Global Vars */
	typedef enum {
		LEFT,
		RIGHT,
		UP,
		DOWN
	} direction_type;

	typedef struct point {
		int x;
		int y;
	} point;

	int x = 0,
		y = 0,
		maxY = 0, 
		maxX = 0,
		nextX = 0,
		nextY = 0,
		tailLength = 5,
		score = 0;

	bool gameOver = false;

	direction_type currentDir = RIGHT;
	point snakeParts[255] = {};
	point food;

/* Functions */
	void createFood() {
		//Food.x is a random int between 10 and maxX - 10
		food.x = (rand() % (maxX - 20)) + 10;

		//Food.y is a random int between 5 and maxY - 5
		food.y = (rand() % (maxY - 10)) + 5;
	}
	
	void drawPart(point drawPoint) {
		mvprintw(drawPoint.y, drawPoint.x, "o");
	}

	void cursesInit() {
		initscr(); //Initialize the window
		noecho(); //Don't echo keypresses
		keypad(stdscr, TRUE);
		cbreak();
		timeout(TIMEOUT);
		curs_set(FALSE); //Don't display a cursor

		//Global var stdscr is created by the call to initscr()
		getmaxyx(stdscr, maxY, maxX);
	}

	void init() {
		srand(time(NULL));

		currentDir = RIGHT;
		tailLength = 5;
		gameOver = false;
		score = 0;

		clear(); //Clears the screen
		
		//Set the initial snake coords 
		int j = 0;
		for(int i = tailLength; i >= 0; i--) {
			point currPoint;
			
			currPoint.x = i;
			currPoint.y = maxY / 2; //Start mid screen on the y axis

			snakeParts[j] = currPoint;
			j++;
		}


		createFood();

		refresh();
	}

	void shiftSnake() {
		point tmp = snakeParts[tailLength - 1];

		for(int i = tailLength - 1; i > 0; i--) {
			snakeParts[i] = snakeParts[i-1];
		}

		snakeParts[0] = tmp;
	}

	void drawScreen() {
		//Clears the screen - put all draw functions after this
		clear(); 

		//Print game over if gameOver is true
		if(gameOver)
			mvprintw(maxY / 2, maxX / 2, "Game Over!");

		//Draw the snake to the screen
		for(int i = 0; i < tailLength; i++) {
			drawPart(snakeParts[i]);
		}

		//Draw the current food
		drawPart(food);

		//Draw the score
		mvprintw(1, 2, "Score: %i", score);

		//ncurses refresh
		refresh();

		//Delay between movements
		usleep(DELAY); 
	}

+#define SPRESENSE_MAIN_BOARD_UART_DEVICE_PATH "/dev/ttyS0"
+
+int getch_serial(int fd) {
+	int len;
+	int ch = 0;
+
+	len = read(fd, (void*)&ch, 1);
+    if(len > 0)
+    {
+		if (ch == 0x1b) {
+			read(fd, &ch, 1);
+			if (ch == 0x5b) {
+				read(fd, &ch, 1);
+				if (ch == 0x44) ch = KEY_LEFT;
+				if (ch == 0x43) ch = KEY_RIGHT;
+				if (ch == 0x41) ch = KEY_UP;
+				if (ch == 0x42) ch = KEY_DOWN;
+			}
+		}
+    }
+
+	return ch;
+}

/* Main */
	int main(int argc, char *argv[]) {
+		_info("snake main()\n");
+
+		int fd = open(SPRESENSE_MAIN_BOARD_UART_DEVICE_PATH, O_RDWR | O_NONBLOCK);
+		if (fd < 0)
+		{
+			printf("%s open error.\n", SPRESENSE_MAIN_BOARD_UART_DEVICE_PATH);
+			return 1;
+		}

		cursesInit();
		init();

		int ch;
		while(1) {
			//Global var stdscr is created by the call to initscr()
			//This tells us the max size of the terminal window at any given moment
			getmaxyx(stdscr, maxY, maxX);
			
			if(gameOver) {
				sleep(2);
				init();
			}

			/* Input Handler */
-				ch = getch();
+				ch = getch_serial(fd);

				if(( ch=='l' || ch=='L' || ch == KEY_RIGHT) && (currentDir != RIGHT && currentDir != LEFT)) {
					currentDir = RIGHT;
				} else if (( ch=='h' || ch=='H' || ch == KEY_LEFT) && (currentDir != RIGHT && currentDir != LEFT)) {
					currentDir = LEFT;
				} else if((ch=='j' || ch=='J' || ch == KEY_DOWN) && (currentDir != UP && currentDir != DOWN)) {
					currentDir = DOWN;
				} else if((ch=='k' || ch=='K' || ch == KEY_UP) && (currentDir != UP && currentDir != DOWN)) {
					currentDir = UP;
+				} else if ((ch=='q' || ch=='Q')) {
+					break;
				}

+				_info("ch = getch() char:%c, hex:%x, dec:%d\n", ch, ch, ch);

			/* Movement */
				nextX = snakeParts[0].x;
				nextY = snakeParts[0].y;

				if(currentDir == RIGHT) nextX++;
		    	else if(currentDir == LEFT) nextX--;
				else if(currentDir == UP) nextY--;
				else if(currentDir == DOWN) nextY++;

				if(nextX == food.x && nextY == food.y) {
					point tail;
					tail.x = nextX;
					tail.y = nextY;

					snakeParts[tailLength] = tail;

					if(tailLength < 255)
						tailLength++;
					else
						tailLength = 5; //If we have exhausted the array then just reset the tail length but let the player keep building their score :)

					score += 5;
					createFood();
				} else {
					//Draw the snake to the screen
					for(int i = 0; i < tailLength; i++) {
						if(nextX == snakeParts[i].x && nextY == snakeParts[i].y) {
							gameOver = true;
							break;
						}
					}

					//We are going to set the tail as the new head
					snakeParts[tailLength - 1].x = nextX;
					snakeParts[tailLength - 1].y = nextY;
				}

				//Shift all the snake parts
				shiftSnake();

				//Game Over if the player hits the screen edges
				if((nextX >= maxX || nextX < 0) || (nextY >= maxY || nextY < 0)) {
					gameOver = true;
				}

			/* Draw the screen */
				drawScreen();
		}

		endwin(); //Restore normal terminal behavior
		nocbreak();

		return 0;
	}
```

コード修正が終了したらmake、コードをflashします。
NuttXのプロンプト（nsh）からsnakeとタイプするとsnakeゲームが実行できます。
上下左右キーでsnakeが動きます。qキーでゲームを終了します。

# なぜ簡単にsnakeゲームができたのか考察
オリジナルのsnakeコマンドはncursesを使っています。
ncursesのWindows向け実装PDCursesがNuttX（※）にポーティングされており、今回はPDCursesを使いました（コンフィグレーションでexamples/pdcursesを指定したところ）。

※Spresense SDKはNuttXのうえで動作しています。

以前slコマンドの移植を行いましたが、snakeゲームもncursesを使用していたのでわりと簡単に移植ができました。

slコマンドの移植

https://qiita.com/juraruming/items/fb8cdbc4d4fe6a503a08

