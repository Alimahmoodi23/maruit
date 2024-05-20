# maruit
a new simple game by coding java
import javafx.animation.AnimationTimer;
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.canvas.Canvas;
import javafx.scene.canvas.GraphicsContext;
import javafx.scene.input.KeyCode;
import javafx.scene.layout.StackPane;
import javafx.scene.paint.Color;
import javafx.stage.Stage;

import java.util.ArrayList;
import java.util.Random;

public class SubwayGame extends Application {
    private static final int WIDTH = 800;
    private static final int HEIGHT = 600;
    private static final int PLAYER_SIZE = 40;
    private static final int GROUND_Y = HEIGHT / 2 - PLAYER_SIZE / 2; // Middle of the screen
    private static final int JUMP_HEIGHT = 200;
    private static final int OBSTACLE_WIDTH = 50;
    private static final int OBSTACLE_HEIGHT = 50;
    private static final int COIN_SIZE = 20;
    private static final int COIN_SCORE = 20;
    private static final int OBSTACLE_SPEED = 5;
    private static final int COIN_INTERVAL = 150;

    private double playerY = GROUND_Y;
    private boolean isJumping = false;
    private int score = 0;
    private boolean gameOver = false;

    private ArrayList<Obstacle> obstacles = new ArrayList<>();
    private ArrayList<Coin> coins = new ArrayList<>();

    @Override
    public void start(Stage primaryStage) {
        Canvas canvas = new Canvas(WIDTH, HEIGHT);
        GraphicsContext gc = canvas.getGraphicsContext2D();

        StackPane root = new StackPane();
        root.getChildren().add(canvas);

        Scene scene = new Scene(root);
        scene.setOnKeyPressed(e -> {
            if (!gameOver) {
                if (e.getCode() == KeyCode.SPACE && !isJumping) {
                    isJumping = true;
                }
                if (e.getCode() == KeyCode.UP) {
                    if (playerY > 0) {
                        playerY -= 40; // Increase movement speed when going up
                    }
                }
                if (e.getCode() == KeyCode.DOWN) {
                    if (playerY < HEIGHT - PLAYER_SIZE) {
                        playerY += 40; // Increase movement speed when going down
                    }
                }
            } else {
                if (e.getCode() == KeyCode.ENTER) {
                    resetGame();
                }
            }
        });

        primaryStage.setScene(scene);
        primaryStage.setTitle("mariut");
        primaryStage.show();

        new AnimationTimer() {
            int coinTimer = 0;

            @Override
            public void handle(long now) {
                gc.clearRect(0, 0, WIDTH, HEIGHT);

                if (!gameOver) {
                    // Draw player
                    gc.setFill(Color.BLUE);
                    gc.fillRect(50, playerY, PLAYER_SIZE, PLAYER_SIZE);

                    // Generate new obstacles randomly
                    if (Math.random() < 0.02) {
                        int y = new Random().nextInt(HEIGHT - OBSTACLE_HEIGHT);
                        obstacles.add(new Obstacle(WIDTH, y));
                    }

                    // Generate coins periodically
                    coinTimer++;
                    if (coinTimer >= COIN_INTERVAL) {
                        coinTimer = 0;
                        int y = new Random().nextInt(HEIGHT - COIN_SIZE);
                        coins.add(new Coin(WIDTH, y));
                    }

                    // Draw and move obstacles
                    for (Obstacle obstacle : new ArrayList<>(obstacles)) {
                        obstacle.move();
                        gc.setFill(Color.BROWN);
                        gc.fillRect(obstacle.getX(), obstacle.getY(), OBSTACLE_WIDTH, OBSTACLE_HEIGHT);

                        // Check collision with player
                        if (obstacle.intersects(50, playerY, PLAYER_SIZE, PLAYER_SIZE)) {
                            gameOver = true; // Game over if collided with obstacle
                        }
                    }

                    // Draw and move coins
                    for (Coin coin : new ArrayList<>(coins)) {
                        coin.move();
                        gc.setFill(Color.PURPLE);
                        gc.fillOval(coin.getX(), coin.getY(), COIN_SIZE, COIN_SIZE);

                        // Check collision with player
                        if (coin.intersects(50, playerY, PLAYER_SIZE, PLAYER_SIZE)) {
                            coins.remove(coin);
                            score += COIN_SCORE; // Increase score if collected coin
                        }
                    }

                    // Remove off-screen obstacles and coins
                    obstacles.removeIf(obstacle -> obstacle.getX() + OBSTACLE_WIDTH < 0);
                    coins.removeIf(coin -> coin.getX() + COIN_SIZE < 0);

                    // Display score
                    gc.setFill(Color.BLACK);
                    gc.fillText("Score: " + score, 10, 20);
                } else {
                    // Draw game over message
                    gc.setFill(Color.BLACK);
                    gc.fillText("Game Over! Press Enter to retry", WIDTH / 2 - 150, HEIGHT / 2);
                }
            }
        }.start();
    }

    public static void main(String[] args) {
        launch(args);
    }

    private static class Obstacle {
        private double x;
        private double y;

        public Obstacle(double x, double y) {
            this.x = x;
            this.y = y;
        }

        public void move() {
            x -= OBSTACLE_SPEED;
        }

        public double getX() {
            return x;
        }

        public double getY() {
            return y;
        }

        public boolean intersects(double otherX, double otherY, double width, double height) {
            return x < otherX + width &&
                    x + OBSTACLE_WIDTH > otherX &&
                    y < otherY + height &&
                    y + OBSTACLE_HEIGHT > otherY;
        }
    }

    private static class Coin {
        private double x;
        private double y;

        public Coin(double x, double y) {
            this.x = x;
            this.y = y;
        }

        public void move() {
            x -= OBSTACLE_SPEED;
        }

        public double getX() {
            return x;
        }

        public double getY() {
            return y;
        }

        public boolean intersects(double otherX, double otherY, double width, double height) {
            return x < otherX + width &&
                    x + COIN_SIZE > otherX &&
                    y < otherY + height &&
                    y + COIN_SIZE > otherY;
        }
    }

    private void resetGame() {
        playerY = GROUND_Y;
        isJumping = false;
        score = 0;
        obstacles.clear();
        coins.clear();
        gameOver = false;
    }
}
