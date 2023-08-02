# perl-game
# Import necessary modules
use strict;
use warnings;
use SDL;
use SDLx::App;
use SDL::Image;

# Initializing SDL
SDL::init(SDL_INIT_VIDEO);

# Setting up the game window
my $app = SDLx::App->new(
    title  => '2D Shooter Game',
    width  => 800,
    height => 600,
    depth  => 32,
);

# Loading graphics: Shooter, Bird, Background
my $shooter_surface = SDL::Image::load('shooter.png');
my $bird_surface    = SDL::Image::load('bird.png');
my $background_surface = SDL::Image::load('background.png');

# Setting initial positions and variables
my $shooter_x = 350;
my $shooter_y = 500;
my $bird_x    = 0;
my $bird_y    = 200;
my $bird_speed = 5;
my $bird_counter = 0;
my $shooting = 0;
my $shooting_timer = 0;
my $shots_fired = 0;

# Game loop
while (1) {
    # Handle events
    while (my $event = SDL::Event->new()) {
        # Quit the game if the user closes the window
        last if $event->type == SDL_QUIT;
        # Handle keyboard events to move the shooter
        if ($event->type == SDL_KEYDOWN) {
            my $key = $event->key_sym;
            if ($key == SDLK_LEFT) {
                $shooter_x -= 10;
            }
            elsif ($key == SDLK_RIGHT) {
                $shooter_x += 10;
            }
        }
        # Handle mouse events to shoot
        elsif ($event->type == SDL_MOUSEBUTTONDOWN) {
            $shooting = 1;
        }
    }

    # Moving the bird from left to right
    $bird_x += $bird_speed;
    if ($bird_x > $app->w) {
        $bird_x = 0;
        $bird_counter++;
    }

    # Checking for collisions (if shooter hits the bird)
    my $bird_hitbox = SDL::Rect->new($bird_x, $bird_y, $bird_surface->w, $bird_surface->h);
    my $shooter_hitbox = SDL::Rect->new($shooter_x, $shooter_y, $shooter_surface->w, $shooter_surface->h);
    if ($shooter_hitbox->has_intersection($bird_hitbox)) {
        # Increment shots_fired if the bird was shot down
        $shots_fired++ if $shooting;
        $shooting = 0;
    }

    # Clearing the screen and draw the background
    $app->draw_rect( undef, 0x000000 );
    $app->blit_by( $background_surface, undef, [ 0, 0, 800, 600 ] );

    # Drawing the shooter and the bird
    $app->blit_by( $shooter_surface, undef, [ $shooter_x, $shooter_y, $shooter_surface->w, $shooter_surface->h ] );
    $app->blit_by( $bird_surface, undef, [ $bird_x, $bird_y, $bird_surface->w, $bird_surface->h ] );

    # Updating the screen
    $app->update;

    # Checking the shooting timer (10 seconds) and reset the bird counter
    if ($shooting_timer >= 10000) {
        if ($shots_fired >= $bird_counter) {
            print "You won!\n";
        } else {
            print "You lost!\n";
        }
        last; # Exit the game loop
    }

    # Incrementing the shooting timer and sleep for a short duration
    $shooting_timer += $app->current_time;
    SDL::delay(10);
}

# Quit SDL
SDL::quit;
