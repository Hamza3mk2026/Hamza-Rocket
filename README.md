import pygame
import math

# إعدادات متقدمة

pygame.init()
WIDTH, HEIGHT = 900, 500
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Hamza Rocket - Pro Edition")
clock = pygame.time.Clock()
font = pygame.font.SysFont("Consolas", 24, bold=True)

# الألوان
STADIUM_GREEN = (20, 120, 60); BALL_WHITE = (240, 240, 240)
P1_COLOR = (0, 180, 255); AI_COLOR = (255, 50, 50); BOOST_COLOR = (255, 200, 0)

# نظام الرتب
mmr = 0
ranks = ["Bronze I", "Silver III", "Gold II", "Platinum I", "Diamond III", "Champion"]

def get_rank(points):
    idx = min(points // 100, len(ranks) - 1)
    return ranks[idx]

class GameObject:
    def __init__(self, x, y, w, h):
        self.rect = pygame.Rect(x, y, w, h)
        self.vel = [0, 0]

def reset_match():
    return {
        "p1": GameObject(100, 235, 50, 30),
        "ai": GameObject(750, 235, 50, 30),
        "ball": GameObject(450, 240, 22, 22),
        "p1_score": 0, "ai_score": 0, "boost": 100, "state": "PLAYING"
    }

data = reset_match()
game_mode = "MENU"

def run_physics(obj, friction=0.98):
    obj.rect.x += obj.vel[0]
    obj.rect.y += obj.vel[1]
    obj.vel[0] *= friction
    obj.vel[1] *= friction

running = True
while running:
    screen.fill((10, 10, 10))
    
    for event in pygame.event.get():
        if event.type == pygame.QUIT: running = False
        if event.type == pygame.KEYDOWN and game_mode == "MENU":
            if event.key == pygame.K_1: game_mode = "RANKED"; data = reset_match()
            if event.key == pygame.K_2: game_mode = "TRAIN"; data = reset_match()

    if game_mode == "MENU":
        title = font.render(f"== HAMZA ROCKET == | RANK: {get_rank(mmr)}", True, P1_COLOR)
        hint = font.render("Press [1] Ranked Match | Press [2] Free Play", True, (200, 200, 200))
        screen.blit(title, (WIDTH//2 - 200, 180))
        screen.blit(hint, (WIDTH//2 - 220, 260))

    else:
        # رسم الملعب
        pygame.draw.rect(screen, STADIUM_GREEN, (10, 10, WIDTH-20, HEIGHT-20))
        pygame.draw.circle(screen, (255,255,255), (WIDTH//2, HEIGHT//2), 70, 2)
        pygame.draw.line(screen, (255,255,255), (WIDTH//2, 10), (WIDTH//2, HEIGHT-10), 2)
        # المرامي (الجون)
        pygame.draw.rect(screen, P1_COLOR, (0, 175, 15, 150))
        pygame.draw.rect(screen, AI_COLOR, (WIDTH-15, 175, 15, 150))

        keys = pygame.key.get_pressed()
        is_boosting = keys[pygame.K_LSHIFT] and data["boost"] > 0
        speed_limit = 12 if is_boosting else 7
        
        # تحكم اللاعب
        if keys[pygame.K_UP]: data["p1"].vel[1] = -speed_limit
        if keys[pygame.K_DOWN]: data["p1"].vel[1] = speed_limit
        if keys[pygame.K_LEFT]: data["p1"].vel[0] = -speed_limit
        if keys[pygame.K_RIGHT]: data["p1"].vel[0] = speed_limit
        
        if is_boosting: data["boost"] -= 1.5
        elif data["boost"] < 100: data["boost"] += 0.2

        # ذكاء اصطناعي (AI)
        if game_mode == "RANKED":
            if data["ai"].rect.centery < data["ball"].rect.centery: data["ai"].vel[1] = 5
            else: data["ai"].vel[1] = -5
            if data["ai"].rect.centerx > data["ball"].rect.centerx + 50: data["ai"].vel[0] = -4

        # تحديث الحركة والفيزياء
        for obj in [data["p1"], data["ai"], data["ball"]]:
            run_physics(obj, 0.92 if obj != data["ball"] else 0.99)
            # منع الخروج من الملعب
            obj.rect.clamp_ip(pygame.Rect(15, 15, WIDTH-30, HEIGHT-30))

        # تصادم الكرة مع اللاعب (قوة الضربة تعتمد على السرعة)
        for car in [data["p1"], data["ai"]]:
            if car.rect.colliderect(data["ball"].rect):
                data["ball"].vel[0] = (data["ball"].rect.centerx - car.rect.centerx) * 0.8
                data["ball"].vel[1] = (data["ball"].rect.centery - car.rect.centery) * 0.8

        # تسجيل الأهداف
        if 175 < data["ball"].rect.centery < 325:
            if data["ball"].rect.left <= 15: # هدف للـ AI
                data["ai_score"] += 1; data = {**data, **reset_match(), "ai_score": data["ai_score"], "p1_score": data["p1_score"]}
            if data["ball"].rect.right >= WIDTH-15: # هدف لحمزة
                data["p1_score"] += 1; data = {**data, **reset_match(), "p1_score": data["p1_score"], "ai_score": data["ai_score"]}

        # رسم العناصر
        pygame.draw.rect(screen, BOOST_COLOR if is_boosting else P1_COLOR, data["p1"].rect, border_radius=5)
        pygame.draw.rect(screen, AI_COLOR, data["ai"].rect, border_radius=5)
        pygame.draw.ellipse(screen, BALL_WHITE, data["ball"].rect)
        
        # واجهة المستخدم
        score_text = font.render(f"YOU: {data['p1_score']} | AI: {data['ai_score']} | BOOST: {int(data['boost'])}%", True, (255, 255, 255))
        screen.blit(score_text, (20, 20))
        
        if data["p1_score"] >= 5: mmr += 30; game_mode = "MENU"
        if data["ai_score"] >= 5: mmr -= 20; game_mode = "MENU"
        if keys[pygame.K_ESCAPE]: game_mode = "MENU"

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
# Hamza-Rocket
mini rocket league with fun
