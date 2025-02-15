import customtkinter as ctk
from tkinter import messagebox
import math
import random
import sqlite3


class BaseApp:
    def __init__(self, main, user_id, title="", geometry="400x200"):
        self.window = ctk.CTkToplevel(main) if main != None else ctk.CTk()
        self.window.title(title)
        self.window.geometry(geometry)
        self.window.configure(fg_color="#96c4df")
        self.userid = user_id
        self.db_manager = Database()

        self.label = ctk.CTkLabel(self.window, text=title, fg_color="#96c4df",
                                  text_color="#333333", font=("Helvetica", 20))
        self.label.pack(pady=20)

    def backtomainmenu(self):
        self.window.destroy()
        self.mainmenu = MainMenu(self.window.master, self.userid)
        self.mainmenu.window.deiconify()

    def getuserdata(self):
        try:
            with sqlite3.connect(self.db_manager.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('''
                    SELECT u.username, p.* 
                    FROM users u
                    JOIN player_data p ON u.id = p.user_id
                    WHERE u.id = ?''', (self.userid,))
                return cursor.fetchone()
        except sqlite3.Error as e:
            print(f"Database error: {e}")
            return None


class Database:
    def __init__(self):
        self.db_name = 'user_login.db'

    @staticmethod
    def conversion(password):
        hashvalue = 0
        prime = 31
        salt = 123456789

        for i, char in enumerate(password):
            hashvalue = (hashvalue ^ (ord(char) * prime + salt)) << (i % 5)
            hashvalue = hashvalue & 0xFFFFFFFFFFFFFFFF

        alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
        hashstr = str(hashvalue)
        for i in range(5):
            hashstr += alphabet[(int(hashvalue) + i) % len(alphabet)]

        return hashstr

    def initialize_db(self):
        conn = None
        try:
            conn = sqlite3.connect(self.db_name)
            if not conn:
                raise Exception("Failed to establish database connection")

            cursor = conn.cursor()

            cursor.execute('''
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT NOT NULL UNIQUE,
                password TEXT NOT NULL
            )
            ''')

            cursor.execute('''
            CREATE TABLE IF NOT EXISTS player_data (
                user_id INTEGER PRIMARY KEY,
                chero TEXT,
                celement TEXT,
                chealth TEXT DEFAULT '100', 
                cstrenght TEXT DEFAULT '5',
                hrarity TEXT,
                cweapon TEXT,
                cweapontype TEXT,
                cweapondamage TEXT DEFAULT '5',
                cweaponabilitiy TEXT,
                wrarity TEXT,
                level INTEGER DEFAULT 1,
                currenthealth TEXT DEFAULT '100',
                currentdamage TEXT DEFAULT '25',
                gold INTEGER DEFAULT 0,
                FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
            )
            ''')

            cursor.execute('''
            CREATE TABLE IF NOT EXISTS hero_inventory (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER,
                hero_name TEXT,
                element TEXT,
                health TEXT,
                strength TEXT,
                rarity TEXT,
                level INTEGER DEFAULT 1,
                equipped INTEGER DEFAULT 0,
                FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
            )
            ''')

            cursor.execute('''
            CREATE TABLE IF NOT EXISTS weapon_inventory (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER,
                weapon_name TEXT,
                weapon_type TEXT,
                damage TEXT,
                ability TEXT,
                rarity TEXT,
                equipped INTEGER DEFAULT 0,
                FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
            )
            ''')

            cursor.execute('''
            CREATE TABLE IF NOT EXISTS team_slots (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER,
                slot_number INTEGER,
                hero_id INTEGER,
                FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
                FOREIGN KEY (hero_id) REFERENCES hero_inventory(id)
            )
            ''')

            conn.commit()

        except Exception as e:
            print(f"Database initialization error: {e}")
            if conn:
                conn.rollback()
        finally:
            if conn:
                conn.close()

    def set_current_health(self, user_id, healthvalue):
        self.updatedb('currenthealth', healthvalue, user_id)

    def set_current_damage(self, user_id, damagevalue):
        self.updatedb('currentdamage', damagevalue, user_id)

    def set_hero(self, heroname, user_id):
        self.updatedb('chero', heroname, user_id)

    def set_element(self, elementname, user_id):
        self.updatedb('celement', elementname, user_id)

    def set_health(self, healthvalue, user_id):
        self.updatedb('chealth', healthvalue, user_id)

    def set_strength(self, strengthvalue, user_id):
        self.updatedb('cstrenght', strengthvalue, user_id)

    def set_hero_rarity(self, herorarity, user_id):
        self.updatedb('hrarity', herorarity, user_id)

    def set_weapon(self, weaponname, user_id):
        self.updatedb('cweapon', weaponname, user_id)

    def set_weapon_type(self, weapontype, user_id):
        self.updatedb('cweapontype', weapontype, user_id)

    def set_weapon_damage(self, weapondamage, user_id):
        self.updatedb('cweapondamage', weapondamage, user_id)

    def set_weapon_ability(self, weaponability, user_id):
        self.updatedb('cweaponabilitiy', weaponability, user_id)

    def set_weapon_rarity(self, weaponrarity, user_id):
        self.updatedb('wrarity', weaponrarity, user_id)

    def updatedb(self, field, value, user_id):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute(f'UPDATE player_data SET {field} = ? WHERE user_id = ?', (value, user_id))
                conn.commit()
        except sqlite3.Error as e:
            print(f"Database error: {e}")

    def get_summon(self, user_id):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('''
                    SELECT chero, celement, chealth, cstrenght, hrarity, cweapon, 
                           cweapontype, cweapondamage, cweaponabilitiy, wrarity 
                    FROM player_data 
                    WHERE user_id = ?''', (user_id,))
                return cursor.fetchone()
        except sqlite3.Error as e:
            print(f"Database error: {e}")
            return None

    @staticmethod
    def calculate_health(level, chealth, element):
        healthincrementperlevel = 20
        element_multipliers = {
            "Fire": 1.1, "Water": 1.2, "Earth": 1.3, "Wind": 1.0,
            "Electric": 1.15, "Ice": 1.25, "Light": 1.2, "Dark": 1.1
        }
        multiplier = element_multipliers.get(element, 1.0)
        return int(chealth + (healthincrementperlevel * level) * multiplier)

    def check_db_data(self):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('SELECT * FROM users')
                data = cursor.fetchall()
                print("Current users in database:", data)
        except sqlite3.Error as e:
            print(f"Database error: {e}")

    def get_user_id(self, username):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute("SELECT id FROM users WHERE username = ?", (username,))
                result = cursor.fetchone()
                return result[0] if result else None
        except sqlite3.Error as e:
            print(f"Database error: {e}")
            return None

    def deleter(self):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('DROP TABLE IF EXISTS player_data')
                cursor.execute('DROP TABLE IF EXISTS users')
                cursor.execute('DROP TABLE IF EXISTS hero_inventory')
                cursor.execute('DROP TABLE IF EXISTS weapon_inventory')
                cursor.execute('DROP TABLE IF EXISTS team_slots')
                conn.commit()
        except sqlite3.Error as e:
            print(f"Database error: {e}")

    def get_summon(self, user_id):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('''
                    SELECT chero, celement, chealth, cstrenght, hrarity, cweapon, 
                           cweapontype, cweapondamage, cweaponabilitiy, wrarity 
                    FROM player_data 
                    WHERE user_id = ?''', (user_id,))
                return cursor.fetchone()
        except sqlite3.Error as e:
            print(f"Database error: {e}")
            return None

    def set_level(self, user_id, level):
        self.updatedb('level', level, user_id)

    @staticmethod
    def calculate_damage(user_id, is_ability=False):
        try:
            conn = sqlite3.connect('user_login.db')
            cursor = conn.cursor()
            
            # Get hero's current strength and level
            cursor.execute('SELECT strength, level FROM hero_inventory WHERE user_id = ? AND equipped = 1', (user_id,))
            hero_data = cursor.fetchone()
            hero_strength = int(hero_data[0]) if hero_data else 5
            hero_level = int(hero_data[1]) if hero_data else 1
            
            # Get weapon's damage
            cursor.execute('SELECT damage FROM weapon_inventory WHERE user_id = ? AND equipped = 1', (user_id,))
            weapon_data = cursor.fetchone()
            weapon_damage = int(weapon_data[0]) if weapon_data else 5
            
            # Calculate scaled strength based on hero level
            scaled_strength = int(hero_strength * (1 + (hero_level * 0.1)))
            
            # Calculate base damage
            base_damage = max(25, scaled_strength * weapon_damage)
            
            # Add level scaling
            level_scaling = (1 + (hero_level ** 1.2) / 10)
            
            # Add randomness (90% - 120% of base damage)
            random_factor = random.uniform(0.9, 1.2)
            
            # Critical hit chance (10% chance to double damage)
            crit_multiplier = 2 if random.random() < 0.1 else 1
            
            # Calculate final damage
            damage = int(base_damage * level_scaling * random_factor * crit_multiplier)
            
            # If this is an ability, multiply by 25
            if is_ability:
                damage *= 25
                
            return damage
            
        except Exception as e:
            print(f"Error calculating damage: {e}")
            return 25  # Return minimum damage on error
        finally:
            if 'conn' in locals():
                conn.close()

    @staticmethod
    def calculatehealth(level, basehealth, element):
        try:
            # Convert inputs to appropriate types
            level = int(level) if level else 1
            basehealth = int(basehealth) if basehealth and str(basehealth).isdigit() else 100

            # Get hero level and health from hero inventory
            conn = sqlite3.connect('user_login.db')
            cursor = conn.cursor()

            cursor.execute('SELECT level, health FROM hero_inventory WHERE user_id = ? AND equipped = 1', (1,))
            hero_data = cursor.fetchone()

            if hero_data:
                hero_level = int(hero_data[0])
                hero_base_health = int(hero_data[1])
            else:
                hero_level = 1
                hero_base_health = basehealth

            conn.close()

            # Scale hero's base health with both character and hero level
            char_scaling = 1 + (level * 0.1)  # 10% increase per character level
            hero_scaling = 1 + (hero_level * 0.15)  # 15% increase per hero level
            scaledhealth = hero_base_health * char_scaling * hero_scaling

            # Element bonuses
            elementmultipliers = {
                "Fire": 1.1,  # Fire gets 10% more health
                "Water": 1.2,  # Water gets 20% more health
                "Earth": 1.3,  # Earth gets 30% more health
                "Wind": 1.0,  # Wind no bonus
                "Electric": 1.15,  # Electric gets 15% more health
                "Ice": 1.25,  # Ice gets 25% more health
                "Light": 1.2,  # Light gets 20% more health
                "Dark": 1.1  # Dark gets 10% more health
            }

            # Apply element multiplier if element exists
            multiplier = elementmultipliers.get(element, 1.0)
            finalhealth = int(scaledhealth * multiplier)

            # Ensure minimum health of 100
            return max(100, finalhealth)

        except Exception as e:
            print(f"Health calculation error: {str(e)}")
            return 100  # Return default health on error

    def add_to_hero_inventory(self, user_id, hero_name, element, health, strength, rarity):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('''
                    INSERT INTO hero_inventory (user_id, hero_name, element, health, strength, rarity)
                    VALUES (?, ?, ?, ?, ?, ?)
                ''', (user_id, hero_name, element, health, strength, rarity))
                conn.commit()
        except sqlite3.Error as e:
            print(f"Database error: {e}")

    def add_to_weapon_inventory(self, user_id, weapon_name, weapon_type, damage, ability, rarity):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('''
                    INSERT INTO weapon_inventory (user_id, weapon_name, weapon_type, damage, ability, rarity)
                    VALUES (?, ?, ?, ?, ?, ?)
                ''', (user_id, weapon_name, weapon_type, damage, ability, rarity))
                conn.commit()
        except sqlite3.Error as e:
            print(f"Database error: {e}")

    def equip_hero(self, user_id, hero_id):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                # Unequip all heroes
                cursor.execute('UPDATE hero_inventory SET equipped = 0 WHERE user_id = ?', (user_id,))
                # Equip selected hero
                cursor.execute('UPDATE hero_inventory SET equipped = 1 WHERE id = ? AND user_id = ?',
                               (hero_id, user_id))
                # Get hero details
                cursor.execute('''
                    SELECT hero_name, element, health, strength, rarity 
                    FROM hero_inventory WHERE id = ? AND user_id = ?
                ''', (hero_id, user_id))
                hero = cursor.fetchone()
                if hero:
                    # Update player_data
                    cursor.execute('''
                        UPDATE player_data 
                        SET chero = ?, celement = ?, chealth = ?, cstrenght = ?, hrarity = ?
                        WHERE user_id = ?
                    ''', (*hero, user_id))
                conn.commit()
        except sqlite3.Error as e:
            print(f"Database error: {e}")

    def equip_weapon(self, user_id, weapon_id):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                # Unequip all weapons
                cursor.execute('UPDATE weapon_inventory SET equipped = 0 WHERE user_id = ?', (user_id,))
                # Equip selected weapon
                cursor.execute('UPDATE weapon_inventory SET equipped = 1 WHERE id = ? AND user_id = ?',
                               (weapon_id, user_id))
                # Get weapon details
                cursor.execute('''
                    SELECT weapon_name, weapon_type, damage, ability, rarity 
                    FROM weapon_inventory WHERE id = ? AND user_id = ?
                ''', (weapon_id, user_id))
                weapon = cursor.fetchone()
                if weapon:
                    # Update player_data
                    cursor.execute('''
                        UPDATE player_data 
                        SET cweapon = ?, cweapontype = ?, cweapondamage = ?, cweaponabilitiy = ?, wrarity = ?
                        WHERE user_id = ?
                    ''', (*weapon, user_id))
                conn.commit()
        except sqlite3.Error as e:
            print(f"Database error: {e}")

    def get_hero_inventory(self, user_id):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('''
                    SELECT id, hero_name, element, health, strength, rarity, equipped
                    FROM hero_inventory WHERE user_id = ?
                ''', (user_id,))
                return cursor.fetchall()
        except sqlite3.Error as e:
            print(f"Database error: {e}")
            return []

    def get_weapon_inventory(self, user_id):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('''
                    SELECT id, weapon_name, weapon_type, damage, ability, rarity, equipped
                    FROM weapon_inventory WHERE user_id = ?
                ''', (user_id,))
                return cursor.fetchall()
        except sqlite3.Error as e:
            print(f"Database error: {e}")
            return []

    def update_gold(self, user_id, amount):
        try:
            with sqlite3.connect(self.db_name) as conn:
                cursor = conn.cursor()
                cursor.execute('UPDATE player_data SET gold = gold + ? WHERE user_id = ?',
                               (amount, user_id))
                conn.commit()
                cursor.execute('SELECT gold FROM player_data WHERE user_id = ?', (user_id,))
                return cursor.fetchone()[0]
        except sqlite3.Error as e:
            print(f"Database error: {e}")
            return None


# Create a global instance of Database
db_manager = Database()


class Entrance(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Entrance", "400x600")

        self.openloginb = ctk.CTkButton(self.window, text="Login", fg_color="white", hover_color="#a5cd9d",
                                        text_color="#333333", font=("Arial", 15), command=self.openloginmenu)
        self.openloginb.pack(pady=20)

        self.openentrance = ctk.CTkButton(self.window, text="Register", fg_color="white", hover_color="#a5cd9d",
                                          text_color="#333333", font=("Arial", 15), command=self.openregistermenu)
        self.openentrance.pack(pady=20)

    def openloginmenu(self):
        self.login = LoginMenu(self.window)
        self.window.withdraw()

    def openregistermenu(self):
        self.registermenu = RegisterMenu(self.window, self.userid)
        self.window.withdraw()


class LoginMenu(BaseApp):
    def __init__(self, main):
        super().__init__(main, None, "Login", "800x400")

        self.userl = ctk.CTkLabel(self.window, text="Username", fg_color="#96c4df", text_color="#333333",
                                  font=("Arial", 15))
        self.userl.place(x=100, y=100)

        self.passl = ctk.CTkLabel(self.window, text="Password", fg_color="#96c4df", text_color="#333333",
                                  font=("Arial", 15))
        self.passl.place(x=100, y=150)

        self.usere = ctk.CTkEntry(self.window, font=("Arial", 15))
        self.usere.place(x=240, y=100)

        self.passe = ctk.CTkEntry(self.window, font=("Arial", 15), show="*")
        self.passe.place(x=240, y=150)

        self.loginb = ctk.CTkButton(self.window, text="Login", fg_color="white", hover_color="#a5cd9d",
                                    text_color="#333333", font=("Arial", 15), command=self.checklogin)
        self.loginb.place(x=240, y=200)

        self.back = ctk.CTkButton(self.window, text="Return to entrance page", fg_color="white", hover_color="#a5cd9d",
                                  text_color="#333333", font=("Arial", 15), command=self.backtoentrance)
        self.back.pack(pady=20, padx=20)
        self.back.place(x=265, y=350)

    def checklogin(self):
        conn = None
        try:
            # Input validation with detailed error messages
            username = self.usere.get().strip()
            password = self.passe.get().strip()

            if not username and not password:
                raise ValueError("Both username and password are required")
            elif not username:
                raise ValueError("Username is required")
            elif not password:
                raise ValueError("Password is required")

            # Database connection with error handling
            try:
                conn = sqlite3.connect('user_login.db')
                if not conn:
                    raise ConnectionError("Failed to establish database connection")
                cursor = conn.cursor()
            except sqlite3.Error as e:
                raise ConnectionError(f"Database connection error: {str(e)}")

            try:
                # Parameterized query to prevent SQL injection
                cursor.execute('SELECT id, password FROM users WHERE username = ?', (username,))
                row = cursor.fetchone()
                print(f"Login attempt for username: {username}")

                if row is not None:
                    self.userid, storedhashedvalue = row
                    try:
                        hashedpassword = Database.conversion(password)
                        print(f"User ID: {self.userid}")
                        print(f"Username: {username}")
                        print("Password hashing completed")

                        if storedhashedvalue == hashedpassword:
                            self.username = username
                            print(f"Login successful for user: {username}")
                            self.openmainmenu()
                        else:
                            raise ValueError("Incorrect password")
                    except Exception as e:
                        print(f"Password processing error: {str(e)}")
                        raise ValueError("Error processing password")
                else:
                    raise ValueError("Username not found")

            except sqlite3.Error as e:
                print(f"Database query error: {str(e)}")
                raise

        except ValueError as e:
            print(f"Validation error: {str(e)}")
            messagebox.showerror("Error", str(e))
        except ConnectionError as e:
            print(f"Connection error: {str(e)}")
            messagebox.showerror("Connection Error", str(e))
        except Exception as e:
            print(f"Unexpected error: {str(e)}")
            messagebox.showerror("Error", "An unexpected error occurred")
        finally:
            if conn:
                try:
                    conn.close()
                except Exception as e:
                    print(f"Error closing database connection: {str(e)}")

    def getusername(self):
        return self.username

    def getuser_id(self):
        return self.userid

    def openmainmenu(self):
        if hasattr(self, 'userid') and self.userid is not None:
            self.mainmenu = MainMenu(self.window, self.userid)  # Pass both 'main' and 'user_id' to MainMenu
            self.window.withdraw()  # Hide the login window
        else:
            messagebox.showerror("Error", "User ID not available.")

    def backtoentrance(self):
        self.window.destroy()
        self.window.master.deiconify()


class RegisterMenu(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Register", "800x400")

        self.userl = ctk.CTkLabel(self.window, text="Username", fg_color="#96c4df", font=("Arial", 15))
        self.userl.place(x=100, y=100)

        self.passl = ctk.CTkLabel(self.window, text="Password", fg_color="#96c4df", font=("Arial", 15))
        self.passl.place(x=100, y=150)

        self.usere = ctk.CTkEntry(self.window, font=("Arial", 15))
        self.usere.place(x=240, y=100)

        self.passe = ctk.CTkEntry(self.window, font=("Arial", 15), show="*")
        self.passe.place(x=240, y=150)

        self.registerb = ctk.CTkButton(self.window, text="Create account", fg_color="white", hover_color="#a5cd9d",
                                       text_color="#333333", font=("Arial", 15), command=self.createaccount)
        self.registerb.place(x=240, y=200)

        self.back = ctk.CTkButton(self.window, text="Return to entrance page", fg_color="white",
                                  hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                  command=self.backtoentrance)
        self.back.pack(pady=20, padx=20)
        self.back.place(x=265, y=350)

    def openmainmenu(self):
        self.mainmenu = MainMenu(self.window, self.userid)
        self.window.withdraw()

    def backtoentrance(self):
        self.window.destroy()
        self.window.master.deiconify()

    def createaccount(self):
        username = self.usere.get()
        password = self.passe.get()

        if not username or not password:
            print("Account creation failed: Empty username or password")
            messagebox.showerror("Input Error", "Username and Password cannot be empty!")
            return

        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()
        try:
            cursor.execute('''
            SELECT u.*, p.* FROM users u 
            LEFT JOIN player_data p ON u.id = p.user_id 
            WHERE u.username = ?
        ''', (username,))
            if cursor.fetchone():
                messagebox.showerror("Username already exists!")
            else:
                hashedpassword = Database.conversion(password)  # Hash the password before storing
                # Insert into users table
                # Insert into users table
                cursor.execute('INSERT INTO users (username, password) VALUES (?, ?)',
                               (username, hashedpassword))

                # Get the new user's ID
                user_id = cursor.lastrowid

                # Insert default values into player_data table
                cursor.execute('''
                    INSERT INTO player_data 
                    (user_id, currenthealth, currentdamage, level, chealth, cstrenght) 
                    VALUES (?, ?, ?, ?, ?, ?)
                ''', (user_id, '100', '0', 1, '100', '0'))
                conn.commit()

                # Get the user ID of the newly created account
                cursor.execute('SELECT id FROM users WHERE username = ?', (username,))
                user_id = cursor.fetchone()[0]
                self.userid = user_id  # Update the instance variable

                print(f"Account created successfully! Username: {username}, UserID: {user_id}")
                messagebox.showinfo("Success", "Account created successfully!")
                print(f"New account created: Username is '{username}' and Hashed password is '{hashedpassword}'")

                self.openmainmenu()
        except sqlite3.Error as e:
            messagebox.showerror("Database Error", f"An error occurred: {str(e)}")
        finally:
            conn.close()


class MainMenu(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Main Menu", "400x200")

        self.battle = ctk.CTkButton(self.window, text="Battle", fg_color="white", hover_color="#a5cd9d",
                                    text_color="#333333", font=("Arial", 15), command=self.openbattle)
        self.battle.pack(pady=20)
        self.battle.place(x=50, y=50)

        self.summon = ctk.CTkButton(self.window, text="Summon", fg_color="white", hover_color="#a5cd9d",
                                    text_color="#333333", font=("Arial", 15), command=self.opensummon)
        self.summon.pack(pady=20)
        self.summon.place(x=220, y=50)

        self.settings = ctk.CTkButton(self.window, text="Settings", fg_color="white", hover_color="#a5cd9d",
                                      text_color="#333333", font=("Arial", 15), command=self.opensettings)
        self.settings.pack(pady=20)
        self.settings.place(x=50, y=140)

        self.user = ctk.CTkButton(self.window, text="User", fg_color="white", hover_color="#a5cd9d",
                                  text_color="#333333",
                                  font=("Arial", 15), command=self.openusers)
        self.user.pack(pady=20)

        self.user.place(x=220, y=140)

    def openbattle(self):
        self.window.withdraw()
        self.battle = Battle(self.window, self.userid)

    def opensummon(self):
        self.window.withdraw()
        self.summon = Summoning(self.window, self.userid)

    def opensettings(self):
        self.window.withdraw()
        self.settings = Settings(self.window, self.userid)

    def openusers(self):
        self.window.withdraw()
        self.users = Users(self.window, self.userid)

    def openinventory(self):
        self.window.withdraw()
        self.inventory = Inventory(self.window, self.userid)


class Modmenu(BaseApp):
    def __init__(self, main, user_id, heroes, weapons, heroprob, weaponprob, heroelements, herohealth, herostrenth,
                 weapontypes, weapondmg, weaponabilities, userswindow):
        super().__init__(main, user_id, "Mod Menu", "800x600")
        self.heroes = heroes
        self.weapons = weapons
        self.heroprob = heroprob
        self.weaponprob = weaponprob
        self.heroelements = heroelements
        self.herohealth = herohealth
        self.herostrenth = herostrenth
        self.weapontypes = weapontypes
        self.weapondmg = weapondmg
        self.weaponabilities = weaponabilities
        self.userswindow = userswindow

        # Create main frames
        self.leftframe = ctk.CTkFrame(self.window, fg_color="#96c4df")
        self.leftframe.pack(side="left", fill="y", padx=10, pady=10)

        self.rightframe = ctk.CTkFrame(self.window, fg_color="#96c4df")
        self.rightframe.pack(side="right", fill="y", padx=10, pady=10)

        # Level setting section
        self.levelframe = ctk.CTkFrame(self.leftframe, fg_color="#96c4df")
        self.levelframe.pack(pady=10)

        self.levellabel = ctk.CTkLabel(self.levelframe, text="Set Level:", fg_color="#96c4df", text_color="#333333",
                                       font=("Arial", 15))
        self.levellabel.pack(side="left", padx=5)

        self.levelentry = ctk.CTkEntry(self.levelframe, font=("Arial", 15), width=100)
        self.levelentry.pack(side="left", padx=5)

        self.setlevelbutton = ctk.CTkButton(self.levelframe, text="Set Level", fg_color="white", hover_color="#a5cd9d",
                                            text_color="#333333", font=("Arial", 15), command=self.updatelevel)
        self.setlevelbutton.pack(side="left", padx=5)

        # Gold setting section
        self.goldframe = ctk.CTkFrame(self.leftframe, fg_color="#96c4df")
        self.goldframe.pack(pady=10)

        self.goldlabel = ctk.CTkLabel(self.goldframe, text="Set Gold:", fg_color="#96c4df", text_color="#333333",
                                      font=("Arial", 15))
        self.goldlabel.pack(side="left", padx=5)

        self.goldentry = ctk.CTkEntry(self.goldframe, font=("Arial", 15), width=100)
        self.goldentry.pack(side="left", padx=5)

        self.setgoldbutton = ctk.CTkButton(self.goldframe, text="Set Gold", fg_color="white", hover_color="#a5cd9d",
                                           text_color="#333333", font=("Arial", 15), command=self.updategold)
        self.setgoldbutton.pack(side="left", padx=5)

        # Hero and Weapon sections
        self.heroframe = ctk.CTkFrame(self.rightframe, fg_color="#96c4df")
        self.heroframe.pack(pady=10)

        self.weaponframe = ctk.CTkFrame(self.rightframe, fg_color="#96c4df")
        self.weaponframe.pack(pady=10)

        # Hero list
        self.herolabel = ctk.CTkLabel(self.heroframe, text="Heroes", fg_color="#96c4df", text_color="#333333",
                                      font=("Arial", 15, "bold"))
        self.herolabel.pack(pady=5)

        self.herolistbox = ctk.CTkScrollableFrame(self.heroframe, width=300, height=200)
        self.herolistbox.pack(padx=10, pady=5)

        # Weapon list
        self.weaponlabel = ctk.CTkLabel(self.weaponframe, text="Weapons", fg_color="#96c4df", text_color="#333333",
                                        font=("Arial", 15, "bold"))
        self.weaponlabel.pack(pady=5)

        self.weaponlistbox = ctk.CTkScrollableFrame(self.weaponframe, width=300, height=200)
        self.weaponlistbox.pack(padx=10, pady=5)

        # Populate lists
        self.populatelistbox(self.herolistbox, heroes, heroprob, self.selecthero)
        self.populatelistbox(self.weaponlistbox, weapons, weaponprob, self.selectweapon)

        # Return button at bottom
        self.returnbutton = ctk.CTkButton(self.window, text="Return to User Menu", fg_color="white",
                                          hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                          command=self.returntousers)
        self.returnbutton.pack(side="bottom", pady=10)

    def mergesort(self, arr):
        if len(arr) <= 1:
            return arr

        mid = len(arr) // 2
        left = self.mergesort(arr[:mid])
        right = self.mergesort(arr[mid:])

        return self.merge(left, right)

    def merge(self, left, right):
        result = []
        i = j = 0

        while i < len(left) and j < len(right):
            # Compare probabilities (second element of tuple)
            if left[i][1] > right[j][1]:
                result.append(left[i])
                i += 1
            else:
                result.append(right[j])
                j += 1

        result.extend(left[i:])
        result.extend(right[j:])
        return result

    def populatelistbox(self, listbox, items, probs, command):
        # Create a list of tuples containing (item, prob)
        item_prob_pairs = list(zip(items, probs))

        # Sort using merge sort
        sorted_pairs = self.mergesort(item_prob_pairs)

        # Create buttons in sorted order
        for item, prob in sorted_pairs:
            color = self.getraritycolor(prob)
            button = ctk.CTkButton(listbox, text=item, text_color=color, font=("Arial", 15),
                                   command=lambda i=item: command(i))
            button.pack(pady=5)

    def getraritycolor(self, prob):
        if prob == 20:
            return "green"
        elif prob == 10:
            return "#2542ce"
        elif prob == 5:
            return "purple"
        elif prob == 1:
            return "#000000"
        return "gray"

    def selecthero(self, heroname):
        element = self.heroelements[heroname]
        health = self.herohealth[heroname]
        strength = self.herostrenth[heroname]
        prob = self.heroprob[self.heroes.index(heroname)]
        rarity = self.getraritycolor(prob)

        setchero(heroname, self.userid)
        setcelement(element, self.userid)
        sethrarity(rarity, self.userid)
        setchealth(health, self.userid)
        setcstrenght(strength, self.userid)

        messagebox.showinfo("Hero Selected",
                            f"Hero: {heroname}\nElement: {element}\nHealth: {health}\nStrength: {strength}")

    def selectweapon(self, weaponname):
        weapontype = self.weapontypes[weaponname]
        weapondamage = self.weapondmg[weaponname]
        weaponability = self.weaponabilities[weaponname]
        prob = self.weaponprob[self.weapons.index(weaponname)]
        rarity = self.getraritycolor(prob)

        setcweapon(weaponname, self.userid)
        setcweapontype(weapontype, self.userid)
        setwrarity(rarity, self.userid)
        setcweapondamage(weapondamage, self.userid)
        setcweaponability(weaponability, self.userid)

        messagebox.showinfo("Weapon Selected",
                            f"Weapon: {weaponname}\nType: {weapontype}\nDamage: {weapondamage}\nAbility: {weaponability}")

    def returntousers(self):
        self.window.destroy()
        self.userswindow.window.deiconify()

    def updatelevel(self):
        try:
            newlevel = int(self.levelentry.get())
            if newlevel < 0:
                messagebox.showerror("Error", "Level cannot be negative!")
                return
            setlevel(self.userid, newlevel)
            messagebox.showinfo("Success", f"Level updated to {newlevel}")
        except ValueError:
            messagebox.showerror("Error", "Please enter a valid number!")

    def updategold(self):
        try:
            newgold = int(self.goldentry.get())
            if newgold < 0:
                messagebox.showerror("Error", "Gold cannot be negative!")
                return

            conn = sqlite3.connect('user_login.db')
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET gold = ? WHERE user_id = ?', (newgold, self.userid))
            conn.commit()
            conn.close()

            messagebox.showinfo("Success", f"Gold updated to {newgold}")
        except ValueError:
            messagebox.showerror("Error", "Please enter a valid number!")
        except sqlite3.Error as e:
            messagebox.showerror("Error", f"Database error: {str(e)}")


class Summoning(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Summon", "400x1100")

        # Create frames for organizing content
        button_frame = ctk.CTkFrame(self.window, fg_color="#96c4df")
        button_frame.pack(pady=10)

        info_frame = ctk.CTkFrame(self.window, fg_color="#96c4df")
        info_frame.pack(pady=10)

        # Add buttons to button frame
        self.herob = ctk.CTkButton(button_frame, text="Summon Hero", fg_color="white", hover_color="#a5cd9d",
                                   text_color="#333333", font=("Arial", 15), command=self.summonhero)
        self.herob.pack(side="left", padx=10)

        self.weaponb = ctk.CTkButton(button_frame, text="Summon Weapon", fg_color="white", hover_color="#a5cd9d",
                                     text_color="#333333", font=("Arial", 15), command=self.summonweapon)
        self.weaponb.pack(side="left", padx=10)

        # Create left and right frames for hero and weapon info
        left_frame = ctk.CTkFrame(info_frame, fg_color="#96c4df")
        left_frame.pack(side="left", padx=20)

        right_frame = ctk.CTkFrame(info_frame, fg_color="#96c4df")
        right_frame.pack(side="right", padx=20)

        # Hero info in left frame
        self.heroinfo = ctk.CTkLabel(left_frame, text="HERO INFO", fg_color="#96c4df", text_color="#333333",
                                     font=("Arial", 15))
        self.heroinfo.pack(pady=5)

        self.heroresult = ctk.CTkLabel(left_frame, text="", fg_color="#96c4df", font=("Arial", 15))
        self.heroresult.pack(pady=5)

        self.elementresult = ctk.CTkLabel(left_frame, text="", fg_color="#96c4df", font=("Arial", 15))
        self.elementresult.pack(pady=5)

        self.healthresult = ctk.CTkLabel(left_frame, text="", fg_color="#96c4df", font=("Arial", 15))
        self.healthresult.pack(pady=5)

        self.strenthgresult = ctk.CTkLabel(left_frame, text="", fg_color="#96c4df", font=("Arial", 15))
        self.strenthgresult.pack(pady=5)

        # Weapon info in right frame
        self.weaponinfo = ctk.CTkLabel(right_frame, text="WEAPON INFO", fg_color="#96c4df", text_color="#333333",
                                       font=("Arial", 15))
        self.weaponinfo.pack(pady=5)

        self.weaponresult = ctk.CTkLabel(right_frame, text="", fg_color="#96c4df", font=("Arial", 15))
        self.weaponresult.pack(pady=5)

        self.typeresult = ctk.CTkLabel(right_frame, text="", fg_color="#96c4df", font=("Arial", 15))
        self.typeresult.pack(pady=5)

        self.damageresult = ctk.CTkLabel(right_frame, text="", fg_color="#96c4df", font=("Arial", 15))
        self.damageresult.pack(pady=5)

        self.abilityresult = ctk.CTkLabel(right_frame, text="", fg_color="#96c4df", font=("Arial", 15))
        self.abilityresult.pack(pady=5)

        self.inventory = ctk.CTkButton(self.window, text="Inventory", fg_color="white", hover_color="#a5cd9d",
                                       text_color="#333333", font=("Arial", 15), command=self.openinventory)
        self.inventory.pack(pady=20)

        self.back = ctk.CTkButton(self.window, text="Return to main menu", fg_color="white", hover_color="#a5cd9d",
                                  text_color="#333333", font=("Arial", 15), command=self.backtomainmenu)
        self.back.pack(pady=20)

        self.currentherolabel = ctk.CTkLabel(self.window, text='', fg_color="#96c4df", font=("Arial", 15))
        self.currentherolabel.pack(pady=5)

        self.currentweaponlabel = ctk.CTkLabel(self.window, text='', fg_color="#96c4df", font=("Arial", 15))
        self.currentweaponlabel.pack(pady=5)

        # Initialize hero and weapon data
        self.hero = [
            "Blaze", "Infernia", "Ember", "Ignis", "Vulcan",  # Fire
            "Marina", "Tidal", "Cascade", "Aquaria", "Sirena",  # Water
            "Terra", "Boulder", "Gaia", "Quarrix", "Petrus",  # Earth
            "Zephyr", "Cyclone", "Aeris", "Ventra", "Skylar",  # Wind
            "Volt", "Spark", "Thunderra", "Zappia", "Storme",  # Electric
            "Frost", "Glaciel", "Chilla", "Cryonix", "Shivera",  # Ice
            "Lumina", "Radiant", "Solara", "Aethera", "Halo",  # Light
            "Umbra", "Nyx", "Shadowe", "Noctis", "Eclipse"  # Dark
        ]

        self.heroprob = [1, 1, 20, 10, 5,  # Fire
                         20, 20, 5, 10, 1,  # Water
                         20, 20, 1, 10, 5,  # Earth
                         1, 20, 5, 20, 10,  # Wind
                         5, 10, 1, 20, 20,  # Electric
                         20, 10, 20, 1, 5,  # Ice
                         10, 20, 5, 20, 1,  # Light
                         20, 5, 20, 10, 1]  # Dark

        self.heroelements = {
            "Blaze": "Fire", "Infernia": "Fire", "Ember": "Fire", "Ignis": "Fire", "Vulcan": "Fire",
            "Marina": "Water", "Tidal": "Water", "Cascade": "Water", "Aquaria": "Water", "Sirena": "Water",
            "Terra": "Earth", "Boulder": "Earth", "Gaia": "Earth", "Quarrix": "Earth", "Petrus": "Earth",
            "Zephyr": "Wind", "Cyclone": "Wind", "Aeris": "Wind", "Ventra": "Wind", "Skylar": "Wind",
            "Volt": "Electric", "Spark": "Electric", "Thunderra": "Electric", "Zappia": "Electric",
            "Storme": "Electric",
            "Frost": "Ice", "Glaciel": "Ice", "Chilla": "Ice", "Cryonix": "Ice", "Shivera": "Ice",
            "Lumina": "Light", "Radiant": "Light", "Solara": "Light", "Aethera": "Light", "Halo": "Light",
            "Umbra": "Dark", "Nyx": "Dark", "Shadowe": "Dark", "Noctis": "Dark", "Eclipse": "Dark"
        }

        self.herohealth = {
            "Blaze": "200", "Infernia": "200", "Ember": "100", "Ignis": "150", "Vulcan": "175",  # Fire
            "Marina": "100", "Tidal": "100", "Cascade": "175", "Aquaria": "150", "Sirena": "200",  # Water
            "Terra": "100", "Boulder": "100", "Gaia": "200", "Quarrix": "150", "Petrus": "175",  # Earth
            "Zephyr": "200", "Cyclone": "100", "Aeris": "175", "Ventra": "100", "Skylar": "150",  # Wind
            "Volt": "175", "Spark": "150", "Thunderra": "200", "Zappia": "100", "Storme": "100",  # Electric
            "Frost": "100", "Glaciel": "150", "Chilla": "100", "Cryonix": "200", "Shivera": "175",  # Ice
            "Lumina": "150", "Radiant": "100", "Solara": "175", "Aethera": "100", "Halo": "200",  # Light
            "Umbra": "100", "Nyx": "175", "Shadowe": "100", "Noctis": "150", "Eclipse": "200"  # Dark
        }

        self.herostrenth = {
            "Blaze": "100", "Infernia": "100", "Ember": "25", "Ignis": "50", "Vulcan": "75",  # Fire
            "Marina": "25", "Tidal": "25", "Cascade": "75", "Aquaria": "50", "Sirena": "100",  # Water
            "Terra": "25", "Boulder": "25", "Gaia": "100", "Quarrix": "50", "Petrus": "75",  # Earth
            "Zephyr": "100", "Cyclone": "25", "Aeris": "75", "Ventra": "25", "Skylar": "50",  # Wind
            "Volt": "75", "Spark": "50", "Thunderra": "100", "Zappia": "25", "Storme": "25",  # Electric
            "Frost": "25", "Glaciel": "50", "Chilla": "25", "Cryonix": "100", "Shivera": "75",  # Ice
            "Lumina": "50", "Radiant": "25", "Solara": "75", "Aethera": "25", "Halo": "100",  # Light
            "Umbra": "25", "Nyx": "75", "Shadowe": "25", "Noctis": "50", "Eclipse": "100"  # Dark
        }

        self.weapons = [
            "Excalibur", "Flametongue", "Blade of Inferno", "Molten Edge", "Lava Cleaver",  # Sword
            "Aqua Harpoon", "Poseidon's Spear", "Coral Lance", "Tidal Pike", "Marine Javelin",  # Spear
            "Rock Crusher", "Earthsplitter", "Terra Mace", "Stone Maul", "Boulder Hammer",  # Axe
            "Zephyr Bow", "Storm Bow", "Gale Shooter", "Tempest Quiver", "Windrunner Bow",  # Bow
            "Lightning Rod", "Thunder Staff", "Volt Wand", "Storm Orb", "Electro Scepter",  # Staff
            "Frost Fang", "Ice Fang", "Glacier Blade", "Cryo Dagger", "Shiver Knife",  # Dagger
            "Radiant Shield", "Halo Guard", "Solar Defender", "Light Barrier", "Aether Buckler",  # Shield
            "Umbra Scythe", "Nyx Reaper", "Shadow Blade", "Noctis Saber", "Eclipse Cutter"  # Scythe
        ]

        self.weaponprob = [1, 20, 5, 10, 20,  # Sword
                           20, 1, 10, 5, 20,  # Spear
                           20, 1, 5, 20, 10,  # Axe
                           1, 5, 20, 10, 20,  # Bow
                           20, 20, 10, 5, 1,  # Staff
                           5, 20, 10, 10, 1,  # Dagger
                           20, 1, 5, 10, 20,  # Shield
                           20, 5, 10, 20, 1]  # Scythe

        self.weaponabilities = {
            "Excalibur": "Flame Sever", "Flametongue": "None", "Blade of Inferno": "None", "Molten Edge": "None",
            "Lava Cleaver": "None",
            "Aqua Harpoon": "None", "Poseidon's Spear": "None", "Coral Lance": "None", "Tidal Pike": "None",
            "Marine Javelin": "Tidal Fury",
            "Rock Crusher": "None", "Earthsplitter": "None", "Terra Mace": "None", "Stone Maul": "None",
            "Boulder Hammer": "Rockfall",
            "Zephyr Bow": "None", "Storm Bow": "None", "Gale Shooter": "None", "Tempest Quiver": "None",
            "Windrunner Bow": "Windstrike",
            "Lightning Rod": "None", "Thunder Staff": "None", "Volt Wand": "None", "Storm Orb": "None",
            "Electro Scepter": "Thunderstorm",
            "Frost Fang": "None", "Ice Fang": "None", "Glacier Blade": "None", "Cryo Dagger": "None",
            "Shiver Knife": "Ice Storm",
            "Radiant Shield": "None", "Halo Guard": "None", "Solar Defender": "None", "Light Barrier": "None",
            "Aether Buckler": "Aether Guard",
            "Umbra Scythe": "None", "Nyx Reaper": "None", "Shadow Blade": "None", "Noctis Saber": "None",
            "Eclipse Cutter": "Shadow Cleaver"
        }

        self.weapondmg = {
            "Excalibur": "100", "Flametongue": "25", "Blade of Inferno": "75", "Molten Edge": "50",
            "Lava Cleaver": "25",
            "Aqua Harpoon": "25", "Poseidon's Spear": "75", "Coral Lance": "25", "Tidal Pike": "50",
            "Marine Javelin": "100",
            "Rock Crusher": "25", "Earthsplitter": "75", "Terra Mace": "50", "Stone Maul": "25",
            "Boulder Hammer": "100",
            "Zephyr Bow": "75", "Storm Bow": "50", "Gale Shooter": "25", "Tempest Quiver": "50",
            "Windrunner Bow": "100",
            "Lightning Rod": "50", "Thunder Staff": "75", "Volt Wand": "25", "Storm Orb": "50",
            "Electro Scepter": "100",
            "Frost Fang": "75", "Ice Fang": "50", "Glacier Blade": "25", "Cryo Dagger": "50", "Shiver Knife": "100",
            "Radiant Shield": "75", "Halo Guard": "50", "Solar Defender": "25", "Light Barrier": "50",
            "Aether Buckler": "100",
            "Umbra Scythe": "75", "Nyx Reaper": "50", "Shadow Blade": "25", "Noctis Saber": "50",
            "Eclipse Cutter": "100"
        }

        self.weapontypes = {
            "Excalibur": "Sword", "Flametongue": "Sword", "Blade of Inferno": "Sword", "Molten Edge": "Sword",
            "Lava Cleaver": "Sword",
            "Aqua Harpoon": "Spear", "Poseidon's Spear": "Spear", "Coral Lance": "Spear", "Tidal Pike": "Spear",
            "Marine Javelin": "Spear",
            "Rock Crusher": "Axe", "Earthsplitter": "Axe", "Terra Mace": "Axe", "Stone Maul": "Axe",
            "Boulder Hammer": "Axe",
            "Zephyr Bow": "Bow", "Storm Bow": "Bow", "Gale Shooter": "Bow", "Tempest Quiver": "Bow",
            "Windrunner Bow": "Bow",
            "Lightning Rod": "Staff", "Thunder Staff": "Staff", "Volt Wand": "Staff", "Storm Orb": "Staff",
            "Electro Scepter": "Staff",
            "Frost Fang": "Dagger", "Ice Fang": "Dagger", "Glacier Blade": "Dagger", "Cryo Dagger": "Dagger",
            "Shiver Knife": "Dagger",
            "Radiant Shield": "Shield", "Halo Guard": "Shield", "Solar Defender": "Shield",
            "Light Barrier": "Shield", "Aether Buckler": "Shield",
            "Umbra Scythe": "Scythe", "Nyx Reaper": "Scythe", "Noctis Saber": "Scythe",
            "Eclipse Cutter": "Scythe"
        }

        # Update display with current hero and weapon info
        self.updatedisplay()

    def summonhero(self):
        # Check if player has enough gold
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()
        cursor.execute('SELECT gold FROM player_data WHERE user_id = ?', (self.userid,))
        current_gold = cursor.fetchone()[0]

        if current_gold < 10000:
            messagebox.showerror("Not Enough Gold", "Summoning requires 10,000 gold!")
            conn.close()
            return

        # Deduct gold and proceed with summon
        cursor.execute('UPDATE player_data SET gold = gold - 10000 WHERE user_id = ?', (self.userid,))
        conn.commit()
        conn.close()

        result = random.choices(self.hero, weights=self.heroprob, k=1)[0]
        element = self.heroelements[result]
        health = self.herohealth[result]
        strength = self.herostrenth[result]
        prob = self.heroprob[self.hero.index(result)]
        hrarity = self.rarity(prob)

        # Check if hero already exists in inventory
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('SELECT id, level FROM hero_inventory WHERE user_id = ? AND hero_name = ?',
                           (self.userid, result))
            existing_hero = cursor.fetchone()
            if existing_hero:
                # Get existing hero's level and show scaled stats
                hero_level = existing_hero[1]
                scaled_health = int(int(health) * (1 + (hero_level * 0.1)))
                scaled_strength = int(int(strength) * (1 + (hero_level * 0.1)))
                self.heroresult.configure(text=f"{result}", text_color=hrarity)
                self.elementresult.configure(text=f"Element: {element}", text_color=hrarity)
                self.healthresult.configure(text=f"Health: {scaled_health}", text_color=hrarity)
                self.strenthgresult.configure(text=f"Strength: {scaled_strength}", text_color=hrarity)
                return

        # Add hero to inventory and auto-equip it
        self.db_manager.add_to_hero_inventory(self.userid, result, element, health, strength, hrarity)
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('SELECT MAX(id) FROM hero_inventory WHERE user_id = ?', (self.userid,))
            latest_hero_id = cursor.fetchone()[0]
            if latest_hero_id:
                self.db_manager.equip_hero(self.userid, latest_hero_id)

                # Get hero level and calculate scaled stats
                cursor.execute('SELECT level FROM hero_inventory WHERE id = ?', (latest_hero_id,))
                level_result = cursor.fetchone()
                level = level_result[0] if level_result else 1

                # Calculate scaled stats (10% increase per level)
                scaled_health = int(int(health) * (1 + (level * 0.1)))
                scaled_strength = int(int(strength) * (1 + (level * 0.1)))

        self.heroresult.configure(text=f"{result}", text_color=hrarity)
        self.elementresult.configure(text=f"Element: {element}", text_color=hrarity)
        self.healthresult.configure(text=f"Health: {scaled_health}", text_color=hrarity)
        self.strenthgresult.configure(text=f"Strength: {scaled_strength}", text_color=hrarity)

        if prob == 1:
            self.spinconformation(self.summonhero)

    def summonweapon(self):
        # Check if player has enough gold
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()
        cursor.execute('SELECT gold FROM player_data WHERE user_id = ?', (self.userid,))
        current_gold = cursor.fetchone()[0]

        if current_gold < 10000:
            messagebox.showerror("Not Enough Gold", "Summoning requires 10,000 gold!")
            conn.close()
            return

        # Deduct gold and proceed with summon
        cursor.execute('UPDATE player_data SET gold = gold - 10000 WHERE user_id = ?', (self.userid,))
        conn.commit()
        conn.close()

        result = random.choices(self.weapons, weights=self.weaponprob, k=1)[0]
        weapontype = self.weapontypes[result]
        weapondamage = self.weapondmg[result]
        weaponability = self.weaponabilities[result]
        prob = self.weaponprob[self.weapons.index(result)]
        wrarity = self.rarity(prob)

        # Check if weapon already exists in inventory
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('SELECT id FROM weapon_inventory WHERE user_id = ? AND weapon_name = ?',
                           (self.userid, result))
            if cursor.fetchone():
                return

        # Add weapon to inventory and auto-equip it
        self.db_manager.add_to_weapon_inventory(self.userid, result, weapontype, weapondamage, weaponability, wrarity)
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('SELECT MAX(id) FROM weapon_inventory WHERE user_id = ?', (self.userid,))
            latest_weapon_id = cursor.fetchone()[0]
            if latest_weapon_id:
                self.db_manager.equip_weapon(self.userid, latest_weapon_id)

        self.weaponresult.configure(text=f"{result}", text_color=wrarity)
        self.typeresult.configure(text=f"Type: {weapontype}", text_color=wrarity)
        self.damageresult.configure(text=f"Damage: {weapondamage}", text_color=wrarity)
        self.abilityresult.configure(text=f"Ability: {weaponability}", text_color=wrarity)

        if prob == 1:
            self.spinconformation(self.summonweapon)

    def spinconformation(self, summonconfirmation):
        response = messagebox.askyesno("Lucky summon!", "You got a unique summon! Would you like to summon again?")
        if response:
            summonconfirmation()

    def rarity(self, prob):
        if prob == 20:
            return "green"
        elif prob == 10:
            return "#2542ce"
        elif prob == 5:
            return "purple"
        elif prob == 1:
            return "#000000"

    def openinventory(self):
        self.window.withdraw()
        self.inventory = Inventory(self.window, self.userid)

    def updatedisplay(self):
        try:
            conn = sqlite3.connect('user_login.db')
            cursor = conn.cursor()
            
            # Get equipped hero stats
            cursor.execute('''
                SELECT hero_name, element, health, strength, rarity, level 
                FROM hero_inventory 
                WHERE user_id = ? AND equipped = 1
            ''', (self.userid,))
            hero_result = cursor.fetchone()
            
            # Get equipped weapon stats
            cursor.execute('''
                SELECT weapon_name, weapon_type, damage, ability, rarity
                FROM weapon_inventory
                WHERE user_id = ? AND equipped = 1
            ''', (self.userid,))
            weapon_result = cursor.fetchone()
            
            if hero_result:
                hero_name, element, base_health, base_strength, hrarity, level = hero_result
                level = int(level) if level else 1
                
                # Calculate scaled stats based on level
                scaled_health = int(int(base_health) * (1 + (level * 0.1)))
                scaled_strength = int(int(base_strength) * (1 + (level * 0.1)))
                
                hrarity = hrarity if hrarity else "gray"
                self.heroresult.configure(text=f"{hero_name}", text_color=hrarity)
                self.elementresult.configure(text=f"Element: {element}", text_color=hrarity)
                self.healthresult.configure(text=f"Health: {scaled_health}", text_color=hrarity)
                self.strenthgresult.configure(text=f"Strength: {scaled_strength}", text_color=hrarity)
            else:
                self.heroresult.configure(text="None", text_color="gray")
                self.elementresult.configure(text="Element: None", text_color="gray")
                self.healthresult.configure(text="Health: None", text_color="gray")
                self.strenthgresult.configure(text="Strength: None", text_color="gray")
            
            if weapon_result:
                weapon_name, weapon_type, damage, ability, wrarity = weapon_result
                wrarity = wrarity if wrarity else "gray"
                self.weaponresult.configure(text=f"{weapon_name}", text_color=wrarity)
                self.typeresult.configure(text=f"Type: {weapon_type}", text_color=wrarity)
                self.damageresult.configure(text=f"Damage: {damage}", text_color=wrarity)
                self.abilityresult.configure(text=f"Ability: {ability}", text_color=wrarity)
            else:
                self.weaponresult.configure(text="None", text_color="gray")
                self.typeresult.configure(text="Type: None", text_color="gray")
                self.damageresult.configure(text="Damage: None", text_color="gray")
                self.abilityresult.configure(text="Ability: None", text_color="gray")
                
        except sqlite3.Error as e:
            print(f"Database error: {e}")
        finally:
            if conn:
                conn.close()


class Inventory(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Inventory", "800x700")

        # Create tabs for Heroes and Weapons
        self.tabview = ctk.CTkTabview(self.window)
        self.tabview.pack(pady=20, expand=True, fill="both")

        self.hero_tab = self.tabview.add("Heroes")
        self.weapon_tab = self.tabview.add("Weapons")

        # Create scrollable frames for each tab
        self.hero_frame = ctk.CTkScrollableFrame(self.hero_tab, width=700, height=400)
        self.hero_frame.pack(pady=10, padx=10, expand=True, fill="both")

        self.weapon_frame = ctk.CTkScrollableFrame(self.weapon_tab, width=700, height=400)
        self.weapon_frame.pack(pady=10, padx=10, expand=True, fill="both")

        # Load and display inventory
        self.load_inventory()

        # Return button
        self.back = ctk.CTkButton(self.window, text="Return to main menu",
                                  fg_color="white", hover_color="#a5cd9d",
                                  text_color="#333333", font=("Arial", 15),
                                  command=self.backtomainmenu)
        self.back.pack(pady=20)

    def load_inventory(self):
        # Clear existing items
        for widget in self.hero_frame.winfo_children():
            widget.destroy()
        for widget in self.weapon_frame.winfo_children():
            widget.destroy()

        # Load heroes
        heroes = self.db_manager.get_hero_inventory(self.userid)
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        try:
            for hero in heroes:
                hero_id, name, element, health, strength, rarity, equipped = hero
                frame = ctk.CTkFrame(self.hero_frame)
                frame.pack(pady=5, padx=5, fill="x")

                color = "gray"
                if equipped:
                    color = "green"

                color = self.get_rarity_color(rarity)

                # Get hero level from database
                cursor.execute('SELECT level FROM hero_inventory WHERE id = ?', (hero_id,))
                level_result = cursor.fetchone()
                level = level_result[0] if level_result and level_result[0] is not None else 1

                scaled_health = int(int(health) * (1 + (level * 0.1)))  # 10% increase per level
                scaled_strength = int(int(strength) * (1 + (level * 0.1)))  # 10% increase per level
                info = f"{name} | Lvl {level} | HP: {scaled_health} | STR: {scaled_strength}"
                label = ctk.CTkLabel(frame, text=info, text_color=color)
                label.pack(side="left", padx=5)

                # Calculate level up cost based on current level
                level_cost = 1000 * (level ** 2)  # Exponential cost increase
                cursor.execute('SELECT gold FROM player_data WHERE user_id = ?', (self.userid,))
                current_gold = cursor.fetchone()[0]

                # Set button color based on whether player can afford level up
                button_color = "green" if current_gold >= level_cost else "red"
                level_btn = ctk.CTkButton(frame, text=f"Level Up ({level_cost} gold)", 
                                         width=120, fg_color=button_color,
                                         command=lambda h_id=hero_id, cost=level_cost: self.level_up_hero(h_id, cost))
                level_btn.pack(side="right", padx=5)

                equip_btn = ctk.CTkButton(frame, text="Equip" if not equipped else "Equipped",
                                          width=100, command=lambda h_id=hero_id: self.equip_hero(h_id))
                equip_btn.pack(side="right", padx=50)
        except Exception as e:
            print(f"Error loading heroes: {e}")

        # Load weapons
        weapons = self.db_manager.get_weapon_inventory(self.userid)
        for weapon in weapons:
            weapon_id, name, type_, damage, ability, rarity, equipped = weapon
            frame = ctk.CTkFrame(self.weapon_frame)
            frame.pack(pady=5, padx=5, fill="x")

            color = "gray"
            if equipped:
                color = "green"

            color = self.get_rarity_color(rarity)
            info = f"{name} | {type_} | DMG: {damage} | Ability: {ability}"
            label = ctk.CTkLabel(frame, text=info, text_color=color)
            label.pack(side="left", padx=5)

            equip_btn = ctk.CTkButton(frame, text="Equip" if not equipped else "Equipped",
                                      width=100, command=lambda w_id=weapon_id: self.equip_weapon(w_id))
            equip_btn.pack(side="right", padx=5)

    def equip_hero(self, hero_id):
        self.db_manager.equip_hero(self.userid, hero_id)
        self.load_inventory()

    def equip_weapon(self, weapon_id):
        self.db_manager.equip_weapon(self.userid, weapon_id)
        self.load_inventory()

    def get_rarity_color(self, rarity):
        rarity_colors = {
            "green": "green",
            "#2542ce": "#2542ce",
            "purple": "purple",
            "#000000": "#000000"
        }
        return rarity_colors.get(rarity, "gray")

    def level_up_hero(self, hero_id, cost):
        try:
            conn = sqlite3.connect('user_login.db')
            cursor = conn.cursor()

            # Check if user has enough gold
            cursor.execute('SELECT gold FROM player_data WHERE user_id = ?', (self.userid,))
            current_gold = cursor.fetchone()[0]

            if current_gold < cost:
                messagebox.showerror("Error", "Not enough gold!")
                return

            # Deduct gold and increase hero level
            cursor.execute('UPDATE player_data SET gold = gold - ? WHERE user_id = ?', (cost, self.userid))
            cursor.execute('UPDATE hero_inventory SET level = level + 1 WHERE id = ?', (hero_id,))
            conn.commit()

            # Refresh inventory display
            self.load_inventory()


        except sqlite3.Error as e:
            print(f"Database error: {e}")
            messagebox.showerror("Error", "Failed to level up hero")
        finally:
            if conn:
                conn.close()

    def backtomainmenu(self):
        self.window.destroy()
        self.mainmenu = MainMenu(self.window.master, self.userid)
        self.mainmenu.window.deiconify()


class Settings(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Settings", "400x600")

        # Get user information from database
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('''
                SELECT u.username, p.currentdamage, p.currenthealth, p.level 
                FROM users u 
                JOIN player_data p ON u.id = p.user_id 
                WHERE u.id = ?''', (self.userid,))
            result = cursor.fetchone()

            if result:
                username, currentdamage, currenthealth, level = result

                # Create and pack labels
                self.useridlabel = ctk.CTkLabel(self.window, text=f"User ID: {self.userid}",
                                                fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
                self.useridlabel.pack(pady=10)

            self.usernamelabel = ctk.CTkLabel(self.window, text=f"Username: {username}",
                                              fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
            self.usernamelabel.pack(pady=10)

            self.damagelabel = ctk.CTkLabel(self.window,
                                            text=f"Current Damage: {currentdamage if currentdamage else '0'}",
                                            fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
            self.damagelabel.pack(pady=10)

            self.healthlabel = ctk.CTkLabel(self.window,
                                            text=f"Current Health: {currenthealth if currenthealth else '100'}",
                                            fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
            self.healthlabel.pack(pady=10)

            self.levellabel = ctk.CTkLabel(self.window, text=f"Level: {level if level else '1'}",
                                           fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
            self.levellabel.pack(pady=10)

            # Add base damage label
            # Get base damage info
            cursor.execute('SELECT cstrenght, cweapondamage FROM player_data WHERE user_id = ?', (self.userid,))
            damage_result = cursor.fetchone()
            if damage_result:
                strength, weapon_damage = damage_result
                strength = int(strength) if strength and strength != 'None' else 5
                weapon_damage = int(weapon_damage) if weapon_damage and weapon_damage != 'None' else 5
                base_damage = max(25, strength * weapon_damage)  # Ensure minimum base damage of 25
                self.base_damage_label = ctk.CTkLabel(self.window, text=f"Base Damage: {base_damage}",
                                                      fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
                self.base_damage_label.pack(pady=10)

            # Add base health label
            cursor.execute('SELECT chealth FROM player_data WHERE user_id = ?', (self.userid,))
            health_result = cursor.fetchone()
            if health_result:
                base_health = health_result[0] if health_result[0] and health_result[0] != 'None' else 100
                self.base_health_label = ctk.CTkLabel(self.window, text=f"Base Health: {base_health}",
                                                      fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
                self.base_health_label.pack(pady=10)

            # Add gold label
            cursor.execute('SELECT gold FROM player_data WHERE user_id = ?', (self.userid,))
            gold_result = cursor.fetchone()
            if gold_result:
                gold = gold_result[0] if gold_result[0] else 0
                self.gold_label = ctk.CTkLabel(self.window, text=f"Gold: {gold}",
                                               fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
                self.gold_label.pack(pady=10)

            self.change_username_button = ctk.CTkButton(self.window, text="Change Username", fg_color="white",
                                                        hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                                        command=self.open_username_change)
            self.change_username_button.pack(pady=10)

        self.back = ctk.CTkButton(self.window, text="Return to main menu", fg_color="white",
                                  hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                  command=self.backtomainmenu)
        self.back.pack(pady=20)

    def open_username_change(self):
        self.window.withdraw()  # Hide settings window
        self.change_window = ctk.CTkToplevel(self.window)
        self.change_window.title("Change Username")
        self.change_window.geometry("300x250")
        self.change_window.configure(fg_color="#96c4df")

        # Handle window close event
        self.change_window.protocol("WM_DELETE_WINDOW", self.on_change_window_close)

        # Username entrya
        self.new_username_entry = ctk.CTkEntry(self.change_window, font=("Arial", 15))
        self.new_username_entry.pack(pady=20)

        # Submit button
        self.submit_button = ctk.CTkButton(self.change_window, text="Submit", fg_color="white",
                                           hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                           command=self.change_username)
        self.submit_button.pack(pady=20)

        # Return button
        self.return_button = ctk.CTkButton(self.change_window, text="Return", fg_color="white",
                                           hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                           command=self.on_change_window_close)
        self.return_button.pack(pady=20)

    def on_change_window_close(self):
        self.change_window.destroy()
        self.window.deiconify()  # Show settings window again

    def change_username(self):
        new_username = self.new_username_entry.get()

        # Check if username is empty
        if not new_username:
            messagebox.showerror("Error", "Please enter a username")
            return

        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        # Check if username already exists
        cursor.execute('SELECT id FROM users WHERE username = ?', (new_username,))
        if cursor.fetchone():
            messagebox.showerror("Error", "Username already exists")
            conn.close()
            return

        # Update username
        try:
            cursor.execute('UPDATE users SET username = ? WHERE id = ?', (new_username, self.userid))
            conn.commit()
            self.usernamelabel.configure(text=f"Username: {new_username}")
            messagebox.showinfo("Success", "Username changed successfully")
            self.change_window.destroy()
            self.window.deiconify()  # Show settings window again
        except sqlite3.Error as e:
            messagebox.showerror("Error", f"An error occurred: {str(e)}")
        finally:
            conn.close()


class Users(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "User Information", "400x400")

        if self.userid == 1:
            self.modbutton = ctk.CTkButton(self.window, text="Mod Menu", fg_color="white", hover_color="#a5cd9d",
                                           text_color="#333333", font=("Arial", 15), command=self.openmodmenu)
            self.modbutton.pack(pady=20)

        self.leaderboard = ctk.CTkButton(self.window, text="Leaderboard", fg_color="white", hover_color="#a5cd9d",
                                         text_color="#333333", font=("Arial", 15), command=self.showleaderboard)
        self.leaderboard.pack(pady=20)

        self.back = ctk.CTkButton(self.window, text="Return to main menu", fg_color="white", hover_color="#a5cd9d",
                                  text_color="#333333", font=("Arial", 15), command=self.backtomainmenu)
        self.back.pack(pady=20)

        self.logout = ctk.CTkButton(self.window, text="Log out", fg_color="white", hover_color="#a5cd9d",
                                    text_color="#333333", font=("Arial", 15), command=self.backtoentrance)
        self.logout.pack(pady=20)

    def level_up_hero(self, hero_id, cost):
        try:
            conn = sqlite3.connect('user_login.db')
            cursor = conn.cursor()

            # Check if user has enough gold
            cursor.execute('SELECT gold FROM player_data WHERE user_id = ?', (self.userid,))
            current_gold = cursor.fetchone()[0]

            if current_gold < cost:
                messagebox.showerror("Error", "Not enough gold!")
                return

            # Deduct gold and increase hero level
            cursor.execute('UPDATE player_data SET gold = gold - ? WHERE user_id = ?', (cost, self.userid))
            cursor.execute('UPDATE hero_inventory SET level = level + 1 WHERE id = ?', (hero_id,))
            conn.commit()

            # Refresh inventory display
            self.load_inventory()
            messagebox.showinfo("Success", "Hero leveled up!")

        except sqlite3.Error as e:
            print(f"Database error: {e}")
            messagebox.showerror("Error", "Failed to level up hero")
        finally:
            if conn:
                conn.close()


        self.back = ctk.CTkButton(self.window, text="Return to main menu", fg_color="white", hover_color="#a5cd9d",
                                  text_color="#333333", font=("Arial", 15), command=self.backtomainmenu)
        self.back.pack(pady=20)

        self.logout = ctk.CTkButton(self.window, text="Log out", fg_color="white", hover_color="#a5cd9d",
                                    text_color="#333333", font=("Arial", 15), command=self.backtoentrance)
        self.logout.pack(pady=20)

    def openmodmenu(self):
        if self.userid == 1:
            # Get all the necessary data that was previously in Summoning class
            self.hero = [
                "Blaze", "Infernia", "Ember", "Ignis", "Vulcan",  # Fire
                "Marina", "Tidal", "Cascade", "Aquaria", "Sirena",  # Water
                "Terra", "Boulder", "Gaia", "Quarrix", "Petrus",  # Earth
                "Zephyr", "Cyclone", "Aeris", "Ventra", "Skylar",  # Wind
                "Volt", "Spark", "Thunderra", "Zappia", "Storme",  # Electric
                "Frost", "Glaciel", "Chilla", "Cryonix", "Shivera",  # Ice
                "Lumina", "Radiant", "Solara", "Aethera", "Halo",  # Light
                "Umbra", "Nyx", "Shadowe", "Noctis", "Eclipse"  # Dark
            ]
            self.heroprob = [1, 1, 20, 10, 5,  # Fire
                             20, 20, 5, 10, 1,  # Water
                             20, 20, 1, 10, 5,  # Earth
                             1, 20, 5, 20, 10,  # Wind
                             5, 10, 1, 20, 20,  # Electric
                             20, 10, 20, 1, 5,  # Ice
                             10, 20, 5, 20, 1,  # Light
                             20, 5, 20, 10, 1]  # Dark
            self.heroelements = {
                "Blaze": "Fire", "Infernia": "Fire", "Ember": "Fire", "Ignis": "Fire", "Vulcan": "Fire",
                "Marina": "Water", "Tidal": "Water", "Cascade": "Water", "Aquaria": "Water", "Sirena": "Water",
                "Terra": "Earth", "Boulder": "Earth", "Gaia": "Earth", "Quarrix": "Earth", "Petrus": "Earth",
                "Zephyr": "Wind", "Cyclone": "Wind", "Aeris": "Wind", "Ventra": "Wind", "Skylar": "Wind",
                "Volt": "Electric", "Spark": "Electric", "Thunderra": "Electric", "Zappia": "Electric",
                "Storme": "Electric",
                "Frost": "Ice", "Glaciel": "Ice", "Chilla": "Ice", "Cryonix": "Ice", "Shivera": "Ice",
                "Lumina": "Light", "Radiant": "Light", "Solara": "Light", "Aethera": "Light", "Halo": "Light",
                "Umbra": "Dark", "Nyx": "Dark", "Shadowe": "Dark", "Noctis": "Dark", "Eclipse": "Dark"
            }
            self.herohealth = {
                "Blaze": "200", "Infernia": "200", "Ember": "100", "Ignis": "150", "Vulcan": "175",  # Fire
                "Marina": "100", "Tidal": "100", "Cascade": "175", "Aquaria": "150", "Sirena": "200",  # Water
                "Terra": "100", "Boulder": "100", "Gaia": "200", "Quarrix": "150", "Petrus": "175",  # Earth
                "Zephyr": "200", "Cyclone": "100", "Aeris": "175", "Ventra": "100", "Skylar": "150",  # Wind
                "Volt": "175", "Spark": "150", "Thunderra": "200", "Zappia": "100", "Storme": "100",  # Electric
                "Frost": "100", "Glaciel": "150", "Chilla": "100", "Cryonix": "200", "Shivera": "175",  # Ice
                "Lumina": "150", "Radiant": "100", "Solara": "175", "Aethera": "100", "Halo": "200",  # Light
                "Umbra": "100", "Nyx": "175", "Shadowe": "100", "Noctis": "150", "Eclipse": "200"  # Dark
            }
            self.herostrenth = {
                "Blaze": "100", "Infernia": "100", "Ember": "25", "Ignis": "50", "Vulcan": "75",  # Fire
                "Marina": "25", "Tidal": "25", "Cascade": "75", "Aquaria": "50", "Sirena": "100",  # Water
                "Terra": "25", "Boulder": "25", "Gaia": "100", "Quarrix": "50", "Petrus": "75",  # Earth
                "Zephyr": "100", "Cyclone": "25", "Aeris": "75", "Ventra": "25", "Skylar": "50",  # Wind
                "Volt": "75", "Spark": "50", "Thunderra": "100", "Zappia": "25", "Storme": "25",  # Electric
                "Frost": "25", "Glaciel": "50", "Chilla": "25", "Cryonix": "100", "Shivera": "75",  # Ice
                "Lumina": "50", "Radiant": "25", "Solara": "75", "Aethera": "25", "Halo": "100",  # Light
                "Umbra": "25", "Nyx": "75", "Shadowe": "25", "Noctis": "50", "Eclipse": "100"  # Dark
            }
            self.weapons = [
                "Excalibur", "Flametongue", "Blade of Inferno", "Molten Edge", "Lava Cleaver",  # Sword
                "Aqua Harpoon", "Poseidon's Spear", "Coral Lance", "Tidal Pike", "Marine Javelin",  # Spear
                "Rock Crusher", "Earthsplitter", "Terra Mace", "Stone Maul", "Boulder Hammer",  # Axe
                "Zephyr Bow", "Storm Bow", "Gale Shooter", "Tempest Quiver", "Windrunner Bow",  # Bow
                "Lightning Rod", "Thunder Staff", "Volt Wand", "Storm Orb", "Electro Scepter",  # Staff
                "Frost Fang", "Ice Fang", "Glacier Blade", "Cryo Dagger", "Shiver Knife",  # Dagger
                "Radiant Shield", "Halo Guard", "Solar Defender", "Light Barrier", "Aether Buckler",  # Shield
                "Umbra Scythe", "Nyx Reaper", "Shadow Blade", "Noctis Saber", "Eclipse Cutter"  # Scythe
            ]
            self.weaponprob = [1, 20, 5, 10, 20,  # Sword
                               20, 1, 10, 5, 20,  # Spear
                               20, 1, 5, 20, 10,  # Axe
                               1, 5, 20, 10, 20,  # Bow
                               20, 20, 10, 5, 1,  # Staff
                               5, 20, 10, 10, 1,  # Dagger
                               20, 1, 5, 10, 20,  # Shield
                               20, 5, 10, 20, 1]  # Scythe
            self.weaponabilities = {
                "Excalibur": "Flame Sever", "Flametongue": "None", "Blade of Inferno": "None", "Molten Edge": "None",
                "Lava Cleaver": "None",
                "Aqua Harpoon": "None", "Poseidon's Spear": "None", "Coral Lance": "None", "Tidal Pike": "None",
                "Marine Javelin": "Tidal Fury",
                "Rock Crusher": "None", "Earthsplitter": "None", "Terra Mace": "None", "Stone Maul": "None",
                "Boulder Hammer": "Rockfall",
                "Zephyr Bow": "None", "Storm Bow": "None", "Gale Shooter": "None", "Tempest Quiver": "None",
                "Windrunner Bow": "Windstrike",
                "Lightning Rod": "None", "Thunder Staff": "None", "Volt Wand": "None", "Storm Orb": "None",
                "Electro Scepter": "Thunderstorm",
                "Frost Fang": "None", "Ice Fang": "None", "Glacier Blade": "None", "Cryo Dagger": "None",
                "Shiver Knife": "Ice Storm",
                "Radiant Shield": "None", "Halo Guard": "None", "Solar Defender": "None", "Light Barrier": "None",
                "Aether Buckler": "Aether Guard",
                "Umbra Scythe": "None", "Nyx Reaper": "None", "Shadow Blade": "None", "Noctis Saber": "None",
                "Eclipse Cutter": "Shadow Cleaver"
            }
            self.weapondmg = {
                "Excalibur": "100", "Flametongue": "25", "Blade of Inferno": "75", "Molten Edge": "50",
                "Lava Cleaver": "25",
                "Aqua Harpoon": "25", "Poseidon's Spear": "75", "Coral Lance": "25", "Tidal Pike": "50",
                "Marine Javelin": "100",
                "Rock Crusher": "25", "Earthsplitter": "75", "Terra Mace": "50", "Stone Maul": "25",
                "Boulder Hammer": "100",
                "Zephyr Bow": "75", "Storm Bow": "50", "Gale Shooter": "25", "Tempest Quiver": "50",
                "Windrunner Bow": "100",
                "Lightning Rod": "50", "Thunder Staff": "75", "Volt Wand": "25", "Storm Orb": "50",
                "Electro Scepter": "100",
                "Frost Fang": "75", "Ice Fang": "50", "Glacier Blade": "25", "Cryo Dagger": "50", "Shiver Knife": "100",
                "Radiant Shield": "75", "Halo Guard": "50", "Solar Defender": "25", "Light Barrier": "50",
                "Aether Buckler": "100",
                "Umbra Scythe": "75", "Nyx Reaper": "50", "Shadow Blade": "25", "Noctis Saber": "50",
                "Eclipse Cutter": "100"
            }
            self.weapontypes = {
                "Excalibur": "Sword", "Flametongue": "Sword", "Blade of Inferno": "Sword", "Molten Edge": "Sword",
                "Lava Cleaver": "Sword",
                "Aqua Harpoon": "Spear", "Poseidon's Spear": "Spear", "Coral Lance": "Spear", "Tidal Pike": "Spear",
                "Marine Javelin": "Spear",
                "Rock Crusher": "Axe", "Earthsplitter": "Axe", "Terra Mace": "Axe", "Stone Maul": "Axe",
                "Boulder Hammer": "Axe",
                "Zephyr Bow": "Bow", "Storm Bow": "Bow", "Gale Shooter": "Bow", "Tempest Quiver": "Bow",
                "Windrunner Bow": "Bow",
                "Lightning Rod": "Staff", "Thunder Staff": "Staff", "Volt Wand": "Staff", "Storm Orb": "Staff",
                "Electro Scepter": "Staff",
                "Frost Fang": "Dagger", "Ice Fang": "Dagger", "Glacier Blade": "Dagger", "Cryo Dagger": "Dagger",
                "Shiver Knife": "Dagger",
                "Radiant Shield": "Shield", "Halo Guard": "Shield", "Solar Defender": "Shield",
                "Light Barrier": "Shield", "Aether Buckler": "Shield",
                "Umbra Scythe": "Scythe", "Nyx Reaper": "Scythe", "Noctis Saber": "Scythe",
                "Eclipse Cutter": "Scythe"
            }
            self.window.withdraw()
            self.modmenu = Modmenu(self.window, self.userid, self.hero, self.weapons,
                                   self.heroprob, self.weaponprob, self.heroelements,
                                   self.herohealth, self.herostrenth, self.weapontypes,
                                   self.weapondmg, self.weaponabilities, self)
        else:
            messagebox.showerror("Access Denied", "You do not have permission to access the Mod Menu.")

    def close_leaderboard(self):
        if hasattr(self, 'leaderboard_window'):
            self.leaderboard_window.destroy()

    def backtoentrance(self):
        self.window.destroy()
        if self.window.master:
            self.window.master.destroy()
        entrance = Entrance(None, None)
        entrance.window.mainloop()

    def showleaderboard(self):
        self.leaderboard_window = ctk.CTkToplevel(self.window)
        self.leaderboard_window.title("Leaderboard")
        self.leaderboard_window.geometry("400x600")
        self.leaderboard_window.configure(fg_color="#96c4df")

        # Handle window close event
        self.leaderboard_window.protocol("WM_DELETE_WINDOW", self.close_leaderboard)

        # Create button frame
        button_frame = ctk.CTkFrame(self.leaderboard_window, fg_color="#96c4df")
        button_frame.pack(pady=10)

        # Create scrollable frame for leaderboard entries
        scrollable_frame = ctk.CTkScrollableFrame(self.leaderboard_window, width=350, height=450)
        scrollable_frame.pack(pady=10)

        # Add title
        self.title_label = ctk.CTkLabel(scrollable_frame, text="Top Players",
                                        fg_color="#96c4df", text_color="#333333",
                                        font=("Arial", 20, "bold"))
        self.title_label.pack(pady=10)

        # Create sort options combobox
        sort_options = ["Sort by Damage", "Sort by Health", "Sort by Gold", "Sort by Level"]
        self.sort_combobox = ctk.CTkComboBox(button_frame, values=sort_options,
                                             fg_color="white", text_color="#333333",
                                             font=("Arial", 15),
                                             command=lambda choice: self.update_leaderboard(choice.split()[-1].lower(),
                                                                                            scrollable_frame))
        self.sort_combobox.pack(pady=10)
        self.sort_combobox.set("Sort by Damage")  # Set default value

        # Show damage leaderboard by default
        self.update_leaderboard("damage", scrollable_frame)

        # Create return button at the bottom
        return_button = ctk.CTkButton(self.leaderboard_window, text="Return", fg_color="white",
                                      hover_color="#a5cd9d", text_color="#333333",
                                      font=("Arial", 15), command=self.close_leaderboard)
        return_button.pack(pady=10)

    def update_leaderboard(self, sort_by, frame):
        # Clear previous entries
        for widget in frame.winfo_children():
            if isinstance(widget, ctk.CTkLabel) and widget != self.title_label:
                widget.destroy()

        # Update title and fetch data
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        if sort_by == "damage":
            self.title_label.configure(text="Top Players by Damage")
            cursor.execute('''
                SELECT u.username, p.currentdamage 
                FROM users u 
                JOIN player_data p ON u.id = p.user_id 
                ORDER BY CAST(COALESCE(p.currentdamage, '0') AS INTEGER) DESC
            ''')
            suffix = "damage"
        elif sort_by == "gold":
            self.title_label.configure(text="Top Players by Gold")
            cursor.execute('''
                SELECT u.username, p.gold 
                FROM users u 
                JOIN player_data p ON u.id = p.user_id 
                ORDER BY COALESCE(p.gold, 0) DESC
            ''')
            suffix = "gold"
        elif sort_by == "level":
            self.title_label.configure(text="Top Players by Level")
            cursor.execute('''
                SELECT u.username, p.level 
                FROM users u 
                JOIN player_data p ON u.id = p.user_id 
                ORDER BY COALESCE(p.level, 1) DESC
            ''')
            suffix = "level"
        else:
            self.title_label.configure(text="Top Players by Health")
            cursor.execute('''
                SELECT u.username, p.currenthealth 
                FROM users u 
                JOIN player_data p ON u.id = p.user_id 
                ORDER BY CAST(COALESCE(p.currenthealth, '0') AS INTEGER) DESC
            ''')
            suffix = "health"

        results = cursor.fetchall()
        conn.close()

        # Add entries to leaderboard
        for rank, (username, value) in enumerate(results, 1):
            value = int(value) if value else 0
            entry = ctk.CTkLabel(frame,
                                 text=f"#{rank} - {username}: {value} {suffix}",
                                 fg_color="#96c4df", text_color="#333333",
                                 font=("Arial", 15))
            entry.pack(pady=5)


class TeamManagement(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Team Management", "800x600")

        self.slots_frame = ctk.CTkFrame(self.window, fg_color="#96c4df")
        self.slots_frame.pack(pady=20)

        self.slots = []
        self.current_heroes = []

        # Create three team slots
        for i in range(3):
            slot_frame = ctk.CTkFrame(self.slots_frame, fg_color="white")
            slot_frame.pack(pady=10, padx=20, side="left")

            slot_label = ctk.CTkLabel(slot_frame, text=f"Slot {i + 1}", font=("Arial", 15))
            slot_label.pack(pady=5)

            hero_label = ctk.CTkLabel(slot_frame, text="Empty", font=("Arial", 15))
            hero_label.pack(pady=5)

            select_button = ctk.CTkButton(slot_frame, text="Select Hero",
                                          command=lambda slot=i: self.select_hero(slot))
            select_button.pack(pady=5)

            self.slots.append({"frame": slot_frame, "label": hero_label})

        self.load_team()

        self.back = ctk.CTkButton(self.window, text="Return to Battle", fg_color="white",
                                  hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                  command=self.backtobattle)
        self.back.pack(pady=20)

    def load_team(self):
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            for i in range(3):
                cursor.execute('''
                    SELECT h.hero_name, h.rarity 
                    FROM team_slots t
                    JOIN hero_inventory h ON t.hero_id = h.id
                    WHERE t.user_id = ? AND t.slot_number = ?
                ''', (self.userid, i))
                result = cursor.fetchone()
                if result:
                    hero_name, rarity = result
                    self.slots[i]["label"].configure(text=hero_name, text_color=rarity)

    def select_hero(self, slot):
        selection_window = ctk.CTkToplevel(self.window)
        selection_window.title(f"Select Hero for Slot {slot + 1}")
        selection_window.geometry("400x600")

        hero_frame = ctk.CTkScrollableFrame(selection_window, width=350, height=500)
        hero_frame.pack(pady=20)

        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('''
                SELECT id, hero_name, rarity 
                FROM hero_inventory 
                WHERE user_id = ? AND id NOT IN 
                    (SELECT hero_id FROM team_slots WHERE user_id = ?)
            ''', (self.userid, self.userid))
            heroes = cursor.fetchall()

            for hero_id, name, rarity in heroes:
                hero_button = ctk.CTkButton(hero_frame, text=name,
                                            text_color=rarity,
                                            command=lambda hid=hero_id, n=name, r=rarity:
                                            self.assign_hero(slot, hid, n, r, selection_window))
                hero_button.pack(pady=5)

    def assign_hero(self, slot, hero_id, name, rarity, window):
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('DELETE FROM team_slots WHERE user_id = ? AND slot_number = ?',
                           (self.userid, slot))
            cursor.execute('INSERT INTO team_slots (user_id, slot_number, hero_id) VALUES (?, ?, ?)',
                           (self.userid, slot, hero_id))
            conn.commit()

        self.slots[slot]["label"].configure(text=name, text_color=rarity)
        window.destroy()

    def backtobattle(self):
        self.window.destroy()
        self.window.master.deiconify()


    def backtoentrance(self):
        self.window.destroy()
        if self.window.master:
            self.window.master.destroy()
        entrance = Entrance(None, None)
        entrance.window.mainloop()


class SiegeMenu(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Siege", "400x450")

        self.startbattlebutton = ctk.CTkButton(self.window, text="Start battle", fg_color="white",
                                               hover_color="#a5cd9d",
                                               text_color="#333333", font=("Helvetica", 20),
                                               command=lambda: self.battlebutton(self.levelup, self.playscene))
        self.startbattlebutton.pack(pady=20)

        self.damagetest = ctk.CTkButton(self.window, text="Damage test", fg_color="white", hover_color="#a5cd9d",
                                        text_color="#333333", font=("Helvetica", 20), command=self.calculatedamage)
        self.damagetest.pack(pady=20)

        self.damageresult = ctk.CTkLabel(self.window, text="Not Measured", fg_color="#96c4df", text_color="#333333",
                                         font=("Helvetica", 20))
        self.damageresult.pack(pady=20)

        self.returnbutton = ctk.CTkButton(self.window, text="Return to Battle Menu", fg_color="white",
                                          hover_color="#a5cd9d", text_color="#333333", font=("Helvetica", 20),
                                          command=self.backtobattlemenu)
        self.returnbutton.pack(pady=20)

    def calculatedamage(self):
        damage = Database.calculate_damage(self.userid)
        self.damageresult.configure(text=f"Damage: {damage}")
        setcurrentdamage(self.userid, str(damage))

    def levelup(self, user_id):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        # Fetch the current level
        cursor.execute('SELECT level FROM player_data WHERE user_id = ?', (user_id,))
        level = cursor.fetchone()

        if level is None:  # User not found
            conn.close()
            print("Level not found for user:", user_id)
            return None

        clevel = level[0] if level[0] is not None else 0  # Default to level 0
        nlevel = clevel + 1

        # Update the new level in the database
        cursor.execute('UPDATE player_data SET level = ? WHERE user_id = ?', (nlevel, user_id))
        conn.commit()
        conn.close()

        print(f"You have leveled up and are now level {nlevel}!")
        return nlevel  # Return the new level

    def backtobattlemenu(self):
        self.window.destroy()
        Battle(self.window.master, self.userid)

    def playscene(self):
        self.window.destroy()
        SiegeMenu(self.window.master, self.userid)

    def battlebutton(self, levelup, playscene):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        # Get current level before leveling up
        cursor.execute('SELECT level FROM player_data WHERE user_id = ?', (self.userid,))
        result = cursor.fetchone()
        current_level = result[0] if result else 1

        # Cap level for gold calculation to prevent overflow
        capped_level = min(50, current_level)  # Cap at level 50 for gold calculation

        # Level up
        new_level = levelup(self.userid)

        if new_level:
            # Calculate gold reward based on capped level
            gold_reward = int(30 * (1.2 ** (capped_level - 1)))  # Base reward for siege
            siege_bonus = int(15 * (1.1 ** (capped_level - 1)))  # Siege completion bonus
            total_gold = gold_reward + siege_bonus

            # Update player's gold
            new_gold = update_gold(self.userid, total_gold)

            messagebox.showinfo("Level Up!",
                                f"Level increased to {new_level}\n"
                                f"Gold earned: {total_gold} (Siege reward: {gold_reward} + Bonus: {siege_bonus})\n"
                                f"Total gold: {new_gold}")

        conn.close()
        playscene()


class Battle(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Battles", "400x400")

        self.siegebutton = ctk.CTkButton(self.window, text="Siege", fg_color="white", hover_color="#a5cd9d",
                                         text_color="#333333", font=("Arial", 15), command=self.opensiege)
        self.siegebutton.pack(pady=20)

        self.questbutton = ctk.CTkButton(self.window, text="Quest", fg_color="white", hover_color="#a5cd9d",
                                         text_color="#333333", font=("Arial", 15), command=self.openquest)
        self.questbutton.pack(pady=20)

        self.team_button = ctk.CTkButton(self.window, text="Team", fg_color="white", hover_color="#a5cd9d",
                                         text_color="#333333", font=("Arial", 15), command=self.openteam)
        self.team_button.pack(pady=20)

        self.back_button = ctk.CTkButton(self.window, text="Return to Main Menu", fg_color="white",
                                         hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                         command=self.backtomainmenu)
        self.back_button.pack(pady=20)

    def openteam(self):
        self.window.withdraw()
        TeamManagement(self.window, self.userid)

    def opensiege(self):
        self.window.withdraw()
        SiegeMenu(self.window, self.userid)

    def openquest(self):
        self.window.withdraw()
        Quest(self.window, self.userid)

    def backtomainmenu(self):
        self.window.destroy()
        self.mainmenu = MainMenu(self.window.master, self.userid)
        self.mainmenu.window.deiconify()


class Quest(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Quest", "400x600")
        # Start Battle Button
        self.start_battle_btn = ctk.CTkButton(self.window, text="Start Battle", fg_color="white",
                                               hover_color="#a5cd9d",
                                               text_color="#333333", font=("Arial", 15), command=self.startquest)
        self.start_battle_btn.pack(pady=20)
        # Damage Test Button
        self.damage_test_btn = ctk.CTkButton(self.window, text="Test Damage", fg_color="white",
                                              hover_color="#a5cd9d",
                                              text_color="#333333", font=("Arial", 15), command=self.test_damage)
        self.damage_test_btn.pack(pady=10)
        # Damage Display Label
        self.damage_display_label = ctk.CTkLabel(self.window, text="Damage: ", fg_color="#96c4df",
                                                  text_color="#333333", font=("Arial", 15))
        self.damage_display_label.pack(pady=10)
    def test_damage(self):
        damage = Database.calculate_damage(self.userid)
        self.damage_display_label.configure(text=f"Damage: {damage}")
        setcurrentdamage(self.userid, str(damage))
        
    def startquest(self):
        conn = None
        try:
            # Establish database connection with error handling
            try:
                conn = sqlite3.connect('user_login.db')
                if not conn:
                    raise ConnectionError("Failed to establish database connection")
                cursor = conn.cursor()
            except sqlite3.Error as e:
                raise ConnectionError(f"Database connection error: {str(e)}")

            # Fetch user data with error handling
            try:
                cursor.execute('SELECT level, celement, chealth FROM player_data WHERE user_id = ?', (self.userid,))
                result = cursor.fetchone()

                if not result:
                    raise ValueError(f"No player data found for user ID: {self.userid}")

                level, element, chealth = result

                # Data validation with defaults
                try:
                    level = int(level) if level else 1
                    if level < 1:
                        level = 1
                    element = element if element and element != "None" else "None"
                    chealth = int(chealth) if chealth and int(chealth) > 0 else 100
                except ValueError as e:
                    print(f"Data conversion error: {str(e)}")
                    level, element, chealth = 1, "None", 100

                # Calculate new health
                try:
                    newhealth = calculatehealth(level, chealth, element)
                    if newhealth < 1:
                        raise ValueError("Invalid health calculation result")
                except Exception as e:
                    print(f"Health calculation error: {str(e)}")
                    newhealth = 100

                # Update database with new health
                try:
                    cursor.execute('UPDATE player_data SET currenthealth = ? WHERE user_id = ?',
                                   (newhealth, self.userid))
                    conn.commit()
                except sqlite3.Error as e:
                    conn.rollback()
                    raise Exception(f"Failed to update health: {str(e)}")

                # Fetch updated health
                cursor.execute('SELECT currenthealth FROM player_data WHERE user_id = ?', (self.userid,))
                userhealth = cursor.fetchone()[0]
                userhealth = int(userhealth) if userhealth else 100

                # Create enemies and start battle
                try:
                    enemies = self.createenemies(level)
                    self.window.destroy()
                    Battles(self.window.master, self.userid, userhealth, enemies)
                except Exception as e:
                    raise Exception(f"Failed to start battle: {str(e)}")

            except sqlite3.Error as e:
                raise Exception(f"Database query error: {str(e)}")

        except ConnectionError as e:
            print(f"Connection error: {str(e)}")
            messagebox.showerror("Connection Error", str(e))
        except ValueError as e:
            print(f"Data error: {str(e)}")
            messagebox.showerror("Error", str(e))
        except Exception as e:
            print(f"Quest error: {str(e)}")
            messagebox.showerror("Error", "Failed to start quest")
        finally:
            if conn is not None:
                try:
                    conn.close()
                except Exception as e:
                    print(f"Error closing database connection: {str(e)}")

    def createenemies(self, level):
        elements = ["Fire", "Water", "Earth", "Wind", "Electric", "Ice", "Light", "Dark"]

        # Get player's stats
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()
        cursor.execute('SELECT cstrenght, cweapondamage, currenthealth FROM player_data WHERE user_id = ?',
                       (self.userid,))
        result = cursor.fetchone()
        conn.close()

        # Calculate player's potential
        userstrength = int(result[0]) if result and result[0] else 1
        userweapondamage = int(result[1]) if result and result[1] else 1
        userhealth = int(result[2]) if result and result[2] else 100
        playerdamage = userstrength * userweapondamage

        # Dynamic scaling factors
        base_scale = 0.4 + (level * 0.08)  # More gradual scaling
        scale_factor = min(base_scale, 1.2)  # Cap at 120% for balance

        # Progressive difficulty for each enemy
        enemies = []
        for i in range(3):
            # Health scaling based on player stats and position
            position_mult = 0.8 + (i * 0.2)  # Later enemies are stronger
            base_health = (300 + (playerdamage * 1.5) + (level * 60)) * position_mult
            health_variance = random.randint(0, level * 30)
            enemyhealth = int((base_health + health_variance) * scale_factor)

            # Damage scaling for fair challenge
            base_damage = max(3, int(userhealth * 0.05) + (level * 3))  # 5% of player health + level bonus
            damage_variance = random.randint(0, level * 2)
            enemydamage = int((base_damage + damage_variance) * scale_factor * position_mult)

            # Ensure minimum and maximum values
            enemyhealth = max(100, min(enemyhealth, userhealth * 3))  # Cap health relative to player
            enemydamage = max(5, min(enemydamage, userhealth // 3))  # Cap damage to prevent one-shots

            enemyelement = random.choice(elements)
            enemies.append({
                "health": enemyhealth,
                "max_health": enemyhealth,
                "damage": enemydamage,
                "element": enemyelement
            })
        return enemies

    def backtobattlemenu(self):
        self.window.destroy()
        Battle(self.window.master, self.userid)


class Battles(BaseApp):
    def __init__(self, questwindow, user_id, userhealth, enemies):
        super().__init__(questwindow, user_id, "Battle", "600x700")
        self.userhealth = userhealth
        self.usermaxhealth = userhealth
        self.enemies = enemies
        self.currentenemy = 0
        self.questwindow = questwindow  # Store reference to quest window

        self.userhealthlabel = ctk.CTkLabel(self.window, text=f"Your Health: {self.userhealth}/{self.usermaxhealth}",
                                            fg_color="#96c4df", text_color="#333333", font=("Helvetica", 15))
        self.userhealthlabel.pack(pady=10)

        self.enemyhealthlabels = []

        for i in range(3):
            label = ctk.CTkLabel(self.window, text="", fg_color="#96c4df", text_color="#333333",
                                 font=("Helvetica", 15))
            label.pack(pady=10)
            self.enemyhealthlabels.append(label)

        self.updatehealthlabels()

        # Add damage display label
        self.damagedisplaylabel = ctk.CTkLabel(self.window, text="Calculating damage...", 
                                               fg_color="#96c4df", text_color="#333333",
                                               font=("Helvetica", 15))
        self.damagedisplaylabel.pack(pady=10)
        
        self.attackbutton = ctk.CTkButton(self.window, text="Attack", fg_color="white", hover_color="#a5cd9d",
                                          text_color="#333333", font=("Helvetica", 20), command=self.attack)
        self.attackbutton.pack(pady=20)
        
        # Start damage calculation update
        self.updatedamagedisplay()

        self.returnbutton = ctk.CTkButton(self.window, text="Return to Quest", fg_color="white", hover_color="#a5cd9d",
                                          text_color="#333333", font=("Helvetica", 20), command=self.returntoquest)
        self.returnbutton.pack(pady=20)

        self.abilitycooldown = 30  # 30 second cooldown
        self.abilityready = True
        self.currentcooldown = 0

        # Add ability button and timer label
        self.abilitybutton = ctk.CTkButton(self.window, text="Use Ability", fg_color="purple", hover_color="#9932CC",
                                           text_color="white", font=("Helvetica", 20), command=self.useability,
                                           state="normal")
        self.abilitybutton.pack(pady=10)

        self.timerlabel = ctk.CTkLabel(self.window, text="Ability Ready!", fg_color="#96c4df", text_color="#333333",
                                       font=("Helvetica", 15))
        self.timerlabel.pack(pady=5)

        # Start the timer update
        self.updatetimer()

    def updatehealthlabels(self):
        self.userhealthlabel.configure(text=f"Your Health: {self.userhealth}/{self.usermaxhealth}")
        for i, label in enumerate(self.enemyhealthlabels):
            if i < len(self.enemies):
                enemy = self.enemies[i]
                label.configure(text=f"Enemy {i + 1} Health: {enemy['health']}/{enemy['max_health']}")
            else:
                label.configure(text="No more enemies")

    def updatedamagedisplay(self):
        try:
            conn = sqlite3.connect('user_login.db')
            cursor = conn.cursor()
            
            # Get hero's current strength and level
            cursor.execute('SELECT strength, level FROM hero_inventory WHERE user_id = ? AND equipped = 1', (self.userid,))
            hero_data = cursor.fetchone()
            hero_strength = int(hero_data[0]) if hero_data else 5
            hero_level = int(hero_data[1]) if hero_data else 1
            
            # Get weapon's damage
            cursor.execute('SELECT damage FROM weapon_inventory WHERE user_id = ? AND equipped = 1', (self.userid,))
            weapon_data = cursor.fetchone()
            weapon_damage = int(weapon_data[0]) if weapon_data else 5
            
            # Calculate scaled strength based on hero level
            scaled_strength = int(hero_strength * (1 + (hero_level * 0.1)))
            
            # Calculate base damage
            base_damage = max(25, scaled_strength * weapon_damage)
            
            # Calculate damage range (90%-120% of base)
            min_damage = int(base_damage * 0.9)
            max_damage = int(base_damage * 1.2)
            
            # Display damage range with crit possibility
            self.damagedisplaylabel.configure(text=f"Attack Damage: {min_damage}-{max_damage} (x2 on crit)")
            
        except Exception as e:
            self.damagedisplaylabel.configure(text="Error calculating damage")
        finally:
            if 'conn' in locals():
                conn.close()
            
        # Update every second
        self.window.after(1000, self.updatedamagedisplay)

    def updatetimer(self):
        if not self.abilityready:
            self.currentcooldown -= 0.01
            if self.currentcooldown <= 0:
                self.abilityready = True
                self.abilitybutton.configure(state="normal", fg_color="purple")
                self.timerlabel.configure(text="Ability Ready!")
            else:
                self.timerlabel.configure(text=f"Cooldown: {self.currentcooldown:.1f}s")

        # next update in 500ms
        self.window.after(10, self.updatetimer)

    def useability(self):
        if not self.abilityready or self.currentenemy >= len(self.enemies):
            return

        enemy = self.enemies[self.currentenemy]
        if self.userhealth <= 0:
            messagebox.showerror("Defeat", "You lost the battle. Try again!")
            self.window.destroy()
            self.questwindow.deiconify()
            return

        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()
        cursor.execute('SELECT cstrenght, cweapondamage, celement, level FROM player_data WHERE user_id = ?',
                       (self.userid,))
        result = cursor.fetchone()
        conn.close()

        if result:
            userstrength, userweapondamage, userelement, level = map(str, result)
            level = int(level) if level else 1
            userstrength = int(userstrength) if userstrength and userstrength != 'None' else 5
            userweapondamage = int(userweapondamage) if userweapondamage and userweapondamage != 'None' else 5
            basedamage = max(25, userstrength * userweapondamage)  # Ensure minimum base damage of 25

            # Calculate full damage like in battle method
            # Calculate ability damage using centralized method
            finaldamage = Database.calculate_damage(self.userid, is_ability=True)

            # Deal damage to enemy
            enemy["health"] -= finaldamage
            if enemy["health"] <= 0:
                enemy["health"] = 0
                self.currentenemy += 1

                if self.currentenemy >= len(self.enemies):
                    messagebox.showinfo("Victory!", "You have defeated all enemies!")
                    self.window.destroy()
                    self.questwindow.deiconify()
                    return

            # Enemy's counter-attack
            if enemy["health"] > 0:
                self.userhealth -= enemy["damage"]
                if self.userhealth <= 0:
                    self.userhealth = 0
                    messagebox.showerror("Defeat", "You lost the battle!")
                    self.window.destroy()  # Close battle screen
                    self.questwindow.deiconify()  # Show previous window
                    return

            # Update health displays
            self.updatehealthlabels()

            # Start cooldown
            self.abilityready = False
            self.currentcooldown = self.abilitycooldown
            self.abilitybutton.configure(state="disabled", fg_color="gray")
            messagebox.showinfo("Ability Used!", f"Dealt {finaldamage} damage with ability!")

    def attack(self):
        if self.currentenemy >= len(self.enemies):
            messagebox.showinfo("Victory!", "You have defeated all enemies!")
            self.window.destroy()
            self.questwindow.deiconify()
            return

        enemy = self.enemies[self.currentenemy]
        if self.userhealth <= 0:
            messagebox.showerror("Defeat", "You lost the battle. Try again!")
            self.window.destroy()
            self.questwindow.deiconify()
            return

        self.battle(enemy)



    def battle(self, enemy):
        try:
            # Calculate damage using the centralized method
            userdamage = Database.calculate_damage(self.userid)
            
            # Prevent extreme damage values
            userdamage = max(25, min(userdamage, enemy["max_health"] // 2))
            
            print(f"Battle started. Enemy health: {enemy['health']}, Damage dealt: {userdamage}")
            
            # Apply damage to enemy
            enemy["health"] -= userdamage
            if enemy["health"] <= 0:
                enemy["health"] = 0
                self.currentenemy += 1
                self.updatehealthlabels()

                if self.currentenemy >= len(self.enemies):
                    print(f"Battle Victory! All enemies defeated by user {self.userid}")
                    self.levelupnext(self.userid)
                    self.window.destroy()
                    self.questwindow.deiconify()
                    return

            # Enemy counter-attack if still alive
            if enemy["health"] > 0:
                self.userhealth -= enemy["damage"]
                print(f"Enemy dealt {enemy['damage']} damage. User health: {self.userhealth}")
                if self.userhealth <= 0:
                    self.userhealth = 0
                    print(f"Battle Lost! User {self.userid} was defeated")
                    messagebox.showerror("Defeat", "You lost the battle!")
                    self.window.destroy()
                    Quest(self.questwindow, self.userid)
                    return

            self.updatehealthlabels()
            
        except Exception as e:
            print(f"Battle error: {e}")
            messagebox.showerror("Error", "Battle system error")

        # Remove unnecessary result check and dedent the code
            # User's turn to attack
            enemy["health"] -= userdamage
            if enemy["health"] <= 0:
                enemy["health"] = 0
                self.currentenemy += 1
                self.updatehealthlabels()

                if self.currentenemy >= len(self.enemies):
                    print(f"Battle Victory! All enemies defeated by user {self.userid}")
                    # Level up the player and show combined message
                    self.levelupnext(self.userid)
                    self.window.destroy()
                    self.questwindow.deiconify()
                    return

            # Enemy's turn to attack only if they're alive
            if enemy["health"] > 0:
                self.userhealth -= enemy["damage"]
                print(f"Enemy dealt {enemy['damage']} damage. User health: {self.userhealth}")
                if self.userhealth <= 0:
                    self.userhealth = 0
                    print(f"Battle Lost! User {self.userid} was defeated")
                    messagebox.showerror("Defeat", "You lost the battle!")
                    self.window.destroy()
                    Quest(self.questwindow, self.userid)  # Return to Quest menu
                    return

            self.updatehealthlabels()

    def levelupnext(self, user_id):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        # Get current level and increment
        cursor.execute('SELECT level, celement, chealth FROM player_data WHERE user_id = ?', (user_id,))
        result = cursor.fetchone()
        if result:
            currentlevel, element, base_health = result
            newlevel = (currentlevel or 0) + 1

            # Calculate new health with level bonus
            base_health = int(base_health) if base_health and str(base_health).isdigit() else 100
            element = element if element and element != "None" else "None"
            newhealth = calculatehealth(newlevel, base_health, element)

            # Update level and health
            cursor.execute('UPDATE player_data SET level = ?, currenthealth = ? WHERE user_id = ?',
                           (newlevel, newhealth, user_id))
            conn.commit()

            # Update battle screen
            self.userhealth = newhealth
            self.usermaxhealth = newhealth

            # Generate new enemies for next round
            self.enemies = self.createenemies(newlevel)
            self.currentenemy = 0

            self.updatehealthlabels()

            # Calculate gold reward based on level
            gold_reward = int(50 * (1.2 ** (newlevel - 1)))  # Exponential scaling
            quest_bonus = int(25 * (1.1 ** (newlevel - 1)))  # Additional quest completion bonus
            total_gold = gold_reward + quest_bonus

            # Update player's gold
            new_gold = update_gold(user_id, total_gold)

            messagebox.showinfo("Victory!",
                                f"Congratulations! You defeated all enemies!\n"
                                f"Level increased to {newlevel}\n"
                                f"Health increased to {newhealth}\n"
                                f"Gold earned: {total_gold} (Level reward: {gold_reward} + Quest bonus: {quest_bonus})\n"
                                f"Total gold: {new_gold}")

        conn.close()

    def createenemies(self, level):
        elements = ["Fire", "Water", "Earth", "Wind", "Electric", "Ice", "Light", "Dark"]
        enemies = []
        for i in range(3):  # Always create 3 enemies
            base_health = level * 10
            health_variance = random.randint(0, level * 5)
            enemyhealth = base_health + health_variance
            base_damage = 5  # Base damage of 5
            level_damage = level * 2 + random.randint(0, level)
            enemydamage = max(base_damage, level_damage)
            enemyelement = random.choice(elements)
            enemies.append({
                "health": enemyhealth,
                "max_health": enemyhealth,
                "damage": enemydamage,
                "element": enemyelement
            })
        return enemies

    def returntoquest(self):
        self.window.destroy()
        self.questwindow.deiconify()

    # Helper functions for database operations


def update_gold(user_id, amount):
    db = Database()
    return db.update_gold(user_id, amount)


def setcurrentdamage(user_id, damage):
    db = Database()
    db.set_current_damage(user_id, damage)


def setlevel(user_id, level):
    db = Database()
    db.set_level(user_id, level)


def setchero(hero, user_id):
    db = Database()
    db.set_hero(hero, user_id)


def setcelement(element, user_id):
    db = Database()
    db.set_element(element, user_id)


def sethrarity(rarity, user_id):
    db = Database()
    db.set_hero_rarity(rarity, user_id)


def setchealth(health, user_id):
    db = Database()
    db.set_health(health, user_id)


def setcstrenght(strength, user_id):
    db = Database()
    db.set_strength(strength, user_id)


def setcweapon(weapon, user_id):
    db = Database()
    db.set_weapon(weapon, user_id)


def setcweapontype(weapon_type, user_id):
    db = Database()
    db.set_weapon_type(weapon_type, user_id)


def setwrarity(rarity, user_id):
    db = Database()
    db.set_weapon_rarity(rarity, user_id)


def setcweapondamage(damage, user_id):
    db = Database()
    db.set_weapon_damage(damage, user_id)


def setcweaponability(ability, user_id):
    db = Database()
    db.set_weapon_ability(ability, user_id)


def getcsummon(user_id):
    db = Database()
    return db.get_summon(user_id)


def calculatehealth(level, basehealth, element):
    db = Database()
    return db.calculatehealth(level, basehealth, element)


def logindata_db():
    db = Database()
    db.initialize_db()


if __name__ == "__main__":
    logindata_db()
    db_manager = Database()
    db_manager.check_db_data()

    main = None
    user_id = None
    entrance = Entrance(main, user_id)
    entrance.window.mainloop()
