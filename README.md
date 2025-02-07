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

        self.label = ctk.CTkLabel(self.window, text=title, fg_color="#96c4df", 
                                 text_color="#333333", font=("Helvetica", 20))
        self.label.pack(pady=20)

    def backtomainmenu(self):
        self.window.destroy()
        self.mainmenu = MainMenu(self.window.master, self.userid)
        self.mainmenu.window.deiconify()




def conversion(password):
    hashvalue = 0
    prime = 31
    salt = 123456789  # Using a static salt to enhance the uniqueness

    for i, char in enumerate(password):
        # XOR each character's ASCII with the salt and shift by the index
        hashvalue = (hashvalue ^ (ord(char) * prime + salt)) << (i % 5)  # Bitwise shift to make it more complex
        hashvalue = hashvalue & 0xFFFFFFFFFFFFFFFF  # Keep hash within 64-bit range

    # Add some letters for even more complexity
    alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
    hashstr = str(hashvalue)
    for i in range(5):  # Add random letters to the end
        hashstr += alphabet[(int(hashvalue) + i) % len(alphabet)]

    return hashstr  # Return as stringhash value instead of the function itself


# Create or connect to the database
def logindata_db():
    conn = None
    try:
        conn = sqlite3.connect('user_login.db')
        if not conn:
            raise Exception("Failed to establish database connection")

        cursor = conn.cursor()

        # Create users table for authentication
        try:
            cursor.execute('''
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT NOT NULL UNIQUE,
                password TEXT NOT NULL
            )
            ''')
        except sqlite3.OperationalError as e:
            print(f"Error creating users table: {e}")
            raise

        # Create player_data table for game data
        try:
            cursor.execute('''
            CREATE TABLE IF NOT EXISTS player_data (
                user_id INTEGER PRIMARY KEY,
                chero TEXT,
                celement TEXT,
                chealth TEXT, 
                cstrenght TEXT,
                hrarity TEXT,
                cweapon TEXT,
                cweapontype TEXT,
                cweapondamage TEXT,
                cweaponabilitiy TEXT,
                wrarity TEXT,
                level INTEGER DEFAULT 1,
                currenthealth TEXT,
                currentdamage TEXT,
                gold INTEGER DEFAULT 0,
                FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
            )
            ''')

        except sqlite3.OperationalError as e:
            print(f"Error creating player_data table: {e}")
            raise

        conn.commit()


    except sqlite3.Error as e:
        print(f"SQLite error during database initialization: {str(e)}")
        if conn:
            conn.rollback()
        raise
    except Exception as e:
        print(f"Unexpected error during database initialization: {str(e)}")
        if conn:
            conn.rollback()
        raise
    finally:
        if conn:
            try:
                conn.close()
            except Exception as e:
                print(f"Error closing database connection: {str(e)}")


def setcurrenthealth(user_id, healthvalue):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET currenthealth = ? WHERE user_id = ?', (healthvalue, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setcurrentdamage(user_id, damagevalue):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET currentdamage = ? WHERE user_id = ?', (damagevalue, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setchero(heroname, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET chero = ? WHERE user_id = ?', (heroname, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setcelement(elementname, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET celement = ? WHERE user_id = ?', (elementname, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setchealth(healthvalue, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET chealth = ? WHERE user_id = ?', (healthvalue, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setcstrenght(strengthvalue, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET cstrenght = ? WHERE user_id = ?', (strengthvalue, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def sethrarity(herorarity, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET hrarity = ? WHERE user_id = ?', (herorarity, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setcweapon(weaponname, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET cweapon = ? WHERE user_id = ?', (weaponname, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setcweapontype(weapontype, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET cweapontype = ? WHERE user_id = ?', (weapontype, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setcweapondamage(weapondamage, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET cweapondamage = ? WHERE user_id = ?', (weapondamage, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setcweaponability(weaponability, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET cweaponabilitiy = ? WHERE user_id = ?', (weaponability, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def setwrarity(weaponrarity, user_id):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET wrarity = ? WHERE user_id = ?', (weaponrarity, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def getcsummon(user_id):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('''
        SELECT chero, celement, chealth, cstrenght, hrarity, cweapon, cweapontype, cweapondamage, cweaponabilitiy, wrarity 
        FROM player_data 
        WHERE user_id = ?''',
        (user_id,))
    result = cursor.fetchone()
    conn.close()
    if result:
        return result  # Returns a tuple with hero, element, hero rarity, weapon, weapon type, weapon rarity
    return None


def calculatehealth(level, chealth, element):
    healthincrementperlevel = 20
    element_multipliers = {
        "Fire": 1.1,
        "Water": 1.2,
        "Earth": 1.3,
        "Wind": 1.0,
        "Electric": 1.15,
        "Ice": 1.25,
        "Light": 1.2,
        "Dark": 1.1
    }
    multiplier = element_multipliers.get(element, 1.0)
    return int(chealth + (healthincrementperlevel * level) * multiplier)


def check_db_data():
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM users')
    data = cursor.fetchall()
    print("Current users in database:", data)
    conn.close()


def getuser_id(username):
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute("SELECT id FROM users WHERE username = ?", (username,))
    result = cursor.fetchone()
    conn.close()
    if result:
        return result[0]
    return None


def deleter():
    conn = sqlite3.connect('user_login.db')
    cursor = conn.cursor()
    cursor.execute('DROP TABLE IF EXISTS player_data')
    cursor.execute('DROP TABLE IF EXISTS users')
    conn.commit()
    conn.close()


def setlevel(user_id, level):
    try:
        with sqlite3.connect('user_login.db') as conn:
            cursor = conn.cursor()
            cursor.execute('UPDATE player_data SET level = ? WHERE user_id = ?', (level, user_id))
            conn.commit()
    except sqlite3.Error as e:
        print(f"Database error: {e}")


def calculateabilitydamage(basedamage):
    return basedamage * 25


def getelementbonus(attacker_element, defender_element):
    elementbonus = {
        "Fire": "Ice",
        "Water": "Fire",
        "Earth": "Electric",
        "Wind": "Earth",
        "Electric": "Water",
        "Ice": "Wind",
        "Light": "Dark",
        "Dark": "Light"
    }
    if elementbonus.get(attacker_element) == defender_element:
        return 1.2  # 20% more damage
    return 1.0

def update_gold(user_id, amount):
  try:
      with sqlite3.connect('user_login.db') as conn:
          cursor = conn.cursor()
          cursor.execute('UPDATE player_data SET gold = gold + ? WHERE user_id = ?', (amount, user_id))
          conn.commit()

          # Get new gold amount
          cursor.execute('SELECT gold FROM player_data WHERE user_id = ?', (user_id,))
          new_gold = cursor.fetchone()[0]
          return new_gold
  except sqlite3.Error as e:
      print(f"Database error: {e}")
      return None

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
                        hashedpassword = conversion(password)
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
                hashedpassword = conversion(password)  # Hash the password before storing
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

        self.user = ctk.CTkButton(self.window, text="User", fg_color="white", hover_color="#a5cd9d", text_color="#333333",
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


class Modmenu(BaseApp):
    def __init__(self, main, user_id, heroes, weapons, heroprob, weaponprob, heroelements, herohealth, herostrenth,weapontypes, weapondmg, weaponabilities, userswindow):
        super().__init__(main, user_id, "Mod Menu", "600x400")
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



        # Add level setting frame
        self.levelframe = ctk.CTkFrame(self.window, fg_color="#96c4df")
        self.levelframe.pack(pady=10)

        self.levellabel = ctk.CTkLabel(self.levelframe, text="Set Level:", fg_color="#96c4df", text_color="#333333",
                                       font=("Arial", 15))
        self.levellabel.pack(side="left", padx=5)

        self.levelentry = ctk.CTkEntry(self.levelframe, font=("Arial", 15), width=100)
        self.levelentry.pack(side="left", padx=5)

        self.setlevelbutton = ctk.CTkButton(self.levelframe, text="Set Level", fg_color="white", hover_color="#a5cd9d",
                                            text_color="#333333", font=("Arial", 15), command=self.updatelevel)
        self.setlevelbutton.pack(side="left", padx=5)

        self.herolistbox = ctk.CTkScrollableFrame(self.window, width=250, height=300)
        self.herolistbox.pack(side="left", padx=10, pady=10)

        self.weaponlistbox = ctk.CTkScrollableFrame(self.window, width=250, height=300)
        self.weaponlistbox.pack(side="right", padx=10, pady=10)

        self.populatelistbox(self.herolistbox, heroes, heroprob, self.selecthero)
        self.populatelistbox(self.weaponlistbox, weapons, weaponprob, self.selectweapon)

        self.returnbutton = ctk.CTkButton(self.window, text="Return to User Menu", fg_color="white",
                                          hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                          command=self.returntousers)
        self.returnbutton.pack(pady=10)

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
                         20, 10,20, 1, 5,  # Ice
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

    def rarity(self, prob):
        if prob == 20:
            return "green"
        elif prob == 10:
            return "#2542ce"
        elif prob == 5:
            return "purple"
        elif prob == 1:
            return "#000000"

    def updatedisplay(self):
        result = getcsummon(self.userid)
        if result:
            chero, celement, chealth, cstrenght, hrarity, cweapon, cweapontype, cweapondamage, cweaponability, wrarity = result
            hrarity = hrarity if hrarity else "gray"  # Set default color if None
            wrarity = wrarity if wrarity else "gray"  # Set default color if None
            self.heroresult.configure(text=f"{chero if chero else 'None'}", text_color=hrarity)
            self.elementresult.configure(text=f"Element: {celement if celement else 'None'}", text_color=hrarity)
            self.healthresult.configure(text=f"Health: {chealth if chealth else 'None'}", text_color=hrarity)
            self.strenthgresult.configure(text=f"Strength: {cstrenght if cstrenght else 'None'}", text_color=hrarity)

            self.weaponresult.configure(text=f"{cweapon if cweapon else 'None'}", text_color=wrarity)
            self.typeresult.configure(text=f"Type: {cweapontype if cweapontype else 'None'}", text_color=wrarity)
            self.damageresult.configure(text=f"Damage: {cweapondamage if cweapondamage else 'None'}",
                                        text_color=wrarity)
            self.abilityresult.configure(text=f"Ability: {cweaponability if cweaponability else 'None'}",
                                         text_color=wrarity)
        else:
            # Handle the case where getcsummon() returns None
            self.heroresult.configure(text="None", text_color="gray")
            self.elementresult.configure(text="None", text_color="gray")
            self.healthresult.configure(text="None", text_color="gray")
            self.strenthgresult.configure(text="None", text_color="gray")

            self.weaponresult.configure(text="None", text_color="gray")
            self.typeresult.configure(text="None", text_color="gray")
            self.damageresult.configure(text="None", text_color="gray")
            self.abilityresult.configure(text="None", text_color="gray")

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
        strenght = self.herostrenth[result]
        prob = self.heroprob[self.hero.index(result)]
        hrarity = self.rarity(prob)
        self.heroresult.configure(text=f" {result} ", text_color=hrarity)
        self.elementresult.configure(text=f"Element: {element}", text_color=hrarity)
        self.healthresult.configure(text=f"Health: {health}", text_color=hrarity)
        self.strenthgresult.configure(text=f"Strength: {strenght}", text_color=hrarity)
        setchero(result, self.userid)
        setcelement(element, self.userid)
        sethrarity(hrarity, self.userid)
        setchealth(health, self.userid)
        setcstrenght(strenght, self.userid)
        self.save_summon(result, element, hrarity, is_hero=True)
        self.updatedisplay()

        if prob == 1:
            print(f"UNIQUE SUMMON: {result}")
        else:
            print(f"Summoned Hero: {result} with probability {prob}")

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
        self.weaponresult.configure(text=f" {result}", text_color=wrarity)
        self.typeresult.configure(text=f"Type: {weapontype}", text_color=wrarity)
        self.damageresult.configure(text=f"Damage: {weapondamage}", text_color=wrarity)
        self.abilityresult.configure(text=f"Ability: {weaponability}", text_color=wrarity)
        setcweapon(result, self.userid)
        setcweapontype(weapontype, self.userid)
        setwrarity(wrarity, self.userid)
        setcweapondamage(weapondamage, self.userid)
        setcweaponability(weaponability, self.userid)
        self.save_summon(result, weapontype, wrarity, is_hero=False)
        self.updatedisplay()

        if prob == 1:
            print(f"UNIQUE SUMMON: {result}")
        else:
            print(f"Summoned Weapon: {result} with probability {prob}")

        if prob == 1:
            self.spinconformation(self.summonweapon)

    def save_summon(self, name, element_or_type, rarity, is_hero=True):
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        if is_hero:
            cursor.execute('''
                UPDATE player_data 
                SET chero = ?, celement = ?, chealth = ?, cstrenght = ?, hrarity = ? 
                WHERE user_id = ?''',
                (name, element_or_type, self.herohealth[name], self.herostrenth[name], rarity, self.userid))
        else:
            cursor.execute('''
                UPDATE player_data 
                SET cweapon = ?, cweapontype = ?, cweapondamage = ?, cweaponabilitiy = ?, wrarity = ? 
                WHERE user_id = ?''',
                (name, element_or_type, self.weapondmg[name], self.weaponabilities[name], rarity, self.userid))

        conn.commit()
        conn.close()

    def spinconformation(self, summonconformation):
        response = messagebox.askyesno("Lucky summon!", "You got a unique summon are you sure you want to spin again")
        if response:
            summonconformation()

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

            self.damagelabel = ctk.CTkLabel(self.window, text=f"Current Damage: {currentdamage if currentdamage else '0'}", 
                                          fg_color="#96c4df", text_color="#333333", font=("Arial", 15))
            self.damagelabel.pack(pady=10)

            self.healthlabel = ctk.CTkLabel(self.window, text=f"Current Health: {currenthealth if currenthealth else '100'}", 
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
                strength = int(strength) if strength and strength != 'None' else 10
                weapon_damage = int(weapon_damage) if weapon_damage and weapon_damage != 'None' else 10
                base_damage = strength * weapon_damage
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
                                  text_color="#333333", font=("Arial", 15),
                                  command=lambda: self.backtomainmenu(user_id))
        self.back.pack(pady=20)

        self.logout = ctk.CTkButton(self.window, text="Log out", fg_color="white", hover_color="#a5cd9d",
                                    text_color="#333333", font=("Arial", 15), command=self.backtoentrance)
        self.logout.pack(pady=20)

    def close_leaderboard(self):
        if hasattr(self, 'leaderboard_window'):
            self.leaderboard_window.destroy()

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
                                            command=lambda choice: self.update_leaderboard(choice.split()[-1].lower(), scrollable_frame))
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
                "Lumina": "150", "Radiant": "100", "Solara": "175", "Aethera": "100","Halo": "200",  # Light
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
                "Frost Fang", "Ice Fang", "Glacier Blade", "Cryo Dagger","Shiver Knife",  # Dagger
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

    def backtomainmenu(self, user_id):
        self.window.destroy()
        self.mainmenu = MainMenu(self.window.master, user_id)
        self.mainmenu.window.deiconify()

    def backtoentrance(self):
        self.window.destroy()
        if self.window.master:
            self.window.master.destroy()
        entrance = Entrance(None, None)
        entrance.window.mainloop()


class SiegeMenu(BaseApp):
    def __init__(self, main, user_id):
        super().__init__(main, user_id, "Siege", "400x450")

        self.startbattlebutton = ctk.CTkButton(self.window, text="Start battle", fg_color="white", hover_color="#a5cd9d",
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
        conn = sqlite3.connect('user_login.db')
        cursor = conn.cursor()

        # Fetch cstrenght, cweapondamage, and level for the given user_id
        cursor.execute('''
        SELECT cstrenght, cweapondamage, level FROM player_data WHERE user_id = ?
        ''', (self.userid,))
        result = cursor.fetchone()
        conn.close()

        if result:
            cstrenght, cweapondamage, level = result
            # Convert values to integers
            cstrenght = int(cstrenght) if cstrenght else 0
            cweapondamage = int(cweapondamage) if cweapondamage else 0
            level = int(level) if level else 0

            # Define the formula to calculate damage
            basedamage = cstrenght * cweapondamage
            levelscaling = (1 + (level ** 1.2) / 10)  # Exponential scaling with level
            randomfactor = random.uniform(0.9, 1.2)  # Adds randomness (90% - 120% of calculated value)
            criticalhit = 2 if random.random() < 0.1 else 1  # 10% chance to double damage

            # Final damage calculation
            damage = math.floor(basedamage * levelscaling * randomfactor * criticalhit)

            # Update the damageresult label with the calculated damage
            self.damageresult.configure(text=f"Damage: {damage}")

            # Update currentdamage in the database
            setcurrentdamage(self.userid, str(damage))
        else:
            self.damageresult.configure(text="Damage: Not Measured")

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

        # Level up
        new_level = levelup(self.userid)

        if new_level:
            # Calculate gold reward based on level
            gold_reward = int(30 * (1.2 ** (current_level - 1)))  # Base reward for siege
            siege_bonus = int(15 * (1.1 ** (current_level - 1)))  # Siege completion bonus
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
        super().__init__(main, user_id, "Battles", "400x300")


        self.siegebutton = ctk.CTkButton(self.window, text="Siege", fg_color="white", hover_color="#a5cd9d",
                                         text_color="#333333", font=("Arial", 15), command=self.opensiege)
        self.siegebutton.pack(pady=20)

        self.questbutton = ctk.CTkButton(self.window, text="Quest", fg_color="white", hover_color="#a5cd9d",
                                         text_color="#333333", font=("Arial", 15), command=self.openquest)
        self.questbutton.pack(pady=20)

        self.back_button = ctk.CTkButton(self.window, text="Return to Main Menu", fg_color="white",
                                         hover_color="#a5cd9d", text_color="#333333", font=("Arial", 15),
                                         command=self.backtomainmenu)
        self.back_button.pack(pady=20)

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
        super().__init__(main, user_id, "Quest", "400x300")


        self.label.pack(pady=20)

        self.startquestbutton = ctk.CTkButton(self.window, text="Start Quest", fg_color="white", hover_color="#a5cd9d",
                                              text_color="#333333", font=("Helvetica", 20), command=self.startquest)
        self.startquestbutton.pack(pady=20)

        self.returnbutton = ctk.CTkButton(self.window, text="Return to Battle Menu", fg_color="white",
                                          hover_color="#a5cd9d", text_color="#333333", font=("Helvetica", 20),
                                          command=self.backtobattlemenu)
        self.returnbutton.pack(pady=20)

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
        cursor.execute('SELECT cstrenght, cweapondamage, currenthealth FROM player_data WHERE user_id = ?', (self.userid,))
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
        super().__init__(questwindow, user_id, "Battle", "600x600")
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

        self.attackbutton = ctk.CTkButton(self.window, text="Attack", fg_color="white", hover_color="#a5cd9d",
                                          text_color="#333333", font=("Helvetica", 20), command=self.attack)
        self.attackbutton.pack(pady=20)

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
        cursor.execute('SELECT cstrenght, cweapondamage, celement, level FROM player_data WHERE user_id = ?', (self.userid,))
        result = cursor.fetchone()
        conn.close()

        if result:
            userstrength, userweapondamage, userelement, level = map(str, result)
            level = int(level) if level else 1
            userstrength = int(userstrength) if userstrength and userstrength != 'None' else 10
            userweapondamage = int(userweapondamage) if userweapondamage and userweapondamage != 'None' else 10
            basedamage = userstrength * userweapondamage

            # Calculate full damage like in battle method
            levelscaling = (1 + (level ** 1.2) / 10)
            randomfactor = random.uniform(0.9, 1.2)
            criticalhit = 2 if random.random() < 0.1 else 1
            elementbonus = getelementbonus(userelement, enemy["element"])

            # Calculate normal attack damage first
            normaldamage = int(basedamage * levelscaling * randomfactor * criticalhit * elementbonus)

            # Multiply by 25 for ability damage
            finaldamage = normaldamage * 25

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

    def close_leaderboard(self):
        self.leaderboard_window.destroy()
        del self.leaderboard_window



        enemy = self.enemies[self.currentenemy]
        if self.userhealth <= 0:
            messagebox.showerror("Defeat", "You lost the battle. Try again!")
            self.window.destroy()
            self.questwindow.deiconify()
            return

        self.battle(enemy)

    def battle(self, enemy):
        result = None
        conn = None
        try:
            conn = sqlite3.connect('user_login.db')
            cursor = conn.cursor()

            try:
                cursor.execute('SELECT cstrenght, cweapondamage, celement, level FROM player_data WHERE user_id = ?', (self.userid,))
                result = cursor.fetchone()

                if not result:
                    print(f"Battle error: No player data found for user {self.userid}")
                    messagebox.showerror("Error", "Player data not found")
                    return

                # Ensure none of the values are None
                userstrength, userweapondamage, userelement, level = result
                if any(val is None for val in [userstrength, userweapondamage, userelement, level]):
                    userstrength = userstrength if userstrength is not None else '10'
                    userweapondamage = userweapondamage if userweapondamage is not None else '10'
                    userelement = userelement if userelement is not None else 'None'
                    level = level if level is not None else '1'
                    result = (userstrength, userweapondamage, userelement, level)

                print(f"Battle started with enemy. Enemy health: {enemy['health']}")
                print(f"Player stats loaded - Strength: {result[0]}, Weapon damage: {result[1]}, Element: {result[2]}, Level: {result[3]}")

            except sqlite3.Error as e:
                print(f"Database error during battle: {e}")
                messagebox.showerror("Error", "Could not load player data")
                return

        except Exception as e:
            print(f"Unexpected error in battle: {e}")
            messagebox.showerror("Error", "Battle system error")
        finally:
            if 'conn' in locals():
                conn.close()

        if result:
            userstrength, userweapondamage, userelement, level = result
            userstrength = int(userstrength) if userstrength and userstrength != 'None' else 10
            userweapondamage = int(userweapondamage) if userweapondamage and userweapondamage != 'None' else 10
            level = int(level) if level is not None else 1

            # Calculate base damage
            basedamage = userstrength * userweapondamage

            # Apply level scaling
            levelscaling = (1 + (level ** 1.2) / 10)

            # Add randomness (90% - 120% of calculated value)
            randomfactor = random.uniform(0.9, 1.2)

            # Critical hit chance (10% chance to double damage)
            criticalhit = 2 if random.random() < 0.1 else 1

            # Calculate element bonus
            elementbonus = getelementbonus(userelement, enemy["element"])

            # Enhanced damage calculation with diminishing returns
            base_scaling = math.log(level + 1, 2) * 1.2  # Logarithmic scaling
            diminished_level = min(levelscaling, base_scaling)
            userdamage = int(basedamage * diminished_level * randomfactor * criticalhit * elementbonus)
            # Prevent extreme damage values
            userdamage = max(10, min(userdamage, enemy["max_health"] // 2))

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
                    self.window.destroy()  # Close battle screen
                    self.questwindow.deiconify()  # Show previous window
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
            enemydamage = max(1, level * 2 + random.randint(0, level))
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

    # Create the main main window


if __name__ == "__main__":
    logindata_db()
    check_db_data()

    main = None
    user_id = None
    entrance = Entrance(main, user_id)
    entrance.window.mainloop()
