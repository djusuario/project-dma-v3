import ctypes
import time
import numpy as np
import win32gui
import win32con

PROCESS_NAME = "robloxplayerbeta.exe"
HITBOX_EXPANDER = 30
SILENT_AIM_ENABLED = True
ESP_COLOR = (0, 255, 0)

class DMA_Aimbot:
    def __init__(self):
        self.hwnd = win32gui.GetForegroundWindow()

    def get_process_handle(self):
        try:
            hwnd = win32gui.FindWindow(None, PROCESS_NAME)
            if not hwnd:
                return None
            pid = win32gui.GetWindowThreadProcessId(hwnd)
            return ctypes.windll.kernel32.OpenProcess(0x1F0FFF, False, pid)
        except Exception:
            return None

    def world_to_screen(self, x, y, z):
        screen_x = x * 100 + 500
        screen_y = y * 100 + 300
        return screen_x, screen_y

    def draw_esp(self, x, y):
        hdc = win32gui.GetDC(self.hwnd)
        if hdc:
            pen = win32gui.CreatePen(win32con.PS_SOLID, 2, ESP_COLOR[0] << 16 | ESP_COLOR[1] << 8 | ESP_COLOR[2])
            win32gui.SelectObject(hdc, pen)
            win32gui.Rectangle(hdc, int(x - 10), int(y - 10), int(x + 10), int(y + 10))
            win32gui.DeleteObject(pen)
            win32gui.ReleaseDC(self.hwnd, hdc)

    def calculate_silent_aim(self, enemy_coords):
        x, y, z = enemy_coords
        screen_x, screen_y = self.world_to_screen(x, y, z)
        if HITBOX_EXPANDER > 0:
            screen_x += np.random.uniform(-HITBOX_EXPANDER, HITBOX_EXPANDER)
            screen_y += np.random.uniform(-HITBOX_EXPANDER, HITBOX_EXPANDER)
        return screen_x, screen_y

    def run(self):
        handle = self.get_process_handle()
        if not handle:
            return

        while True:
            enemy_x, enemy_y, enemy_z = 50.0, 10.0, 100.0
            aim_x, aim_y = self.calculate_silent_aim((enemy_x, enemy_y, enemy_z))
            self.draw_esp(aim_x, aim_y)
            if SILENT_AIM_ENABLED:
                pass
            time.sleep(0.016)

if __name__ == "__main__":
    bot = DMA_Aimbot()
    bot.run()
