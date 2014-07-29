import libtcodpy2 as libtcod
import math
import textwrap
import shelve
import weapon_var
import Dungeon_story
import name_fetch
import Kill_calls
#actual size of the window
SCREEN_WIDTH = 100
SCREEN_HEIGHT = 55
 
#size of the map 
MAP_WIDTH = 80
MAP_HEIGHT = 47

PANEL_RIGHT_W = 20

#sizes and coordinates relevant for the GUI
BAR_WIDTH = 18
PANEL_HEIGHT = 8
PANEL_Y = SCREEN_HEIGHT - PANEL_HEIGHT
MSG_X = BAR_WIDTH + 2
MSG_WIDTH = SCREEN_WIDTH - BAR_WIDTH - 22
MSG_HEIGHT = PANEL_HEIGHT - 3
INVENTORY_WIDTH =  50

THROW_RADIUS = 0
THROW_DAMAGE = 2
record_xp = 0 

#parameters for dungeon generator
ROOM_MAX_SIZE = 10
ROOM_MIN_SIZE = 6
MAX_ROOMS = 50

#experience and level-ups
LEVEL_UP_BASE = 200
LEVEL_UP_FACTOR = 100
LEVEL_SCREEN_WIDTH = 40
CHARACTER_SCREEN_WIDTH = 30

#spell values
HEAL_AMOUNT = 4
turn_step = 0
turn_step_level = 0
regen = 0
set_regen = 10

FOV_ALGO = 0  #default FOV algorithm
FOV_LIGHT_WALLS = True
TORCH_RADIUS = 3

LIMIT_FPS = 20  #20 frames-per-second maximum
screen_track = 0

color_dark_wall = libtcod.Color(50, 50, 50)
color_light_wall = libtcod.Color(190, 190, 190)
color_dark_ground = libtcod.Color(35, 35, 35)
color_light_ground = libtcod.Color(140, 80, 25)

inventory = [] 
weight_cap = 10
weight_inventory = 1

#MOB KILLS
warg_kill = 0
warg_pup_kill = 0
rabbit_kill = 0
skeletonnewb_kill = 0
crebain_kill = 0
skeleton_kill = 0
wolf_kill = 0
mewlip_kill = 0
spectre_kill = 0
spectrenewb_kill = 0
greywolf_kill = 0
snaga_kill = 0
snuffler_kill = 0
wargleader_kill = 0

EP_pool = 0
EP_cost_hp = 5
EP_cost_power = 5
EP_cost_defence = 5
EP_cost_Throwpower = 5
EP_cost_endurance = 5
EP_cost_weightcap = 5
EP_cost_vision = 10
EP_cost_luck = 10
Accumulator = 5

display = 0
display_max_hp = 0
display_damage = 0

STUN_NUM_TURNS = 10

take_steps = 0
take_steps_n = 0
take_floor = 0
take_games = 0
small_steps = 10000
big_steps = 0

option_hint = 1
option_fullscreen = 1

######
#Tiles#
######
class Tile:
    #a tile of the map and its properties
    def __init__(self, blocked, block_sight = None):
        self.blocked = blocked
        
        #all tiles start unexplored
        self.explored = False
 
        #by default, if a tile is blocked, it also blocks sight
        if block_sight is None: block_sight = blocked
        self.block_sight = block_sight

################
#DungeonCreator#    
################        
class Rect:
    #a rectangle on the map. used to characterize a room.
    def __init__(self, x, y, w, h):
        self.x1 = x
        self.y1 = y
        self.x2 = x + w
        self.y2 = y + h
        
    def center(self):
        center_x = (self.x1 + self.x2) / 2
        center_y = (self.y1 + self.y2) / 2
        return (center_x, center_y)
 
    def intersect(self, other):
        #returns true if this rectangle intersects with another one
        return (self.x1 <= other.x2 and self.x2 >= other.x1 and
                self.y1 <= other.y2 and self.y2 >= other.y1)
                
########
#Objects#
########

class Object:
    #this is a generic object: the player, a monster, an item, the stairs...
    #it's always represented by a character on screen.
    def __init__(self, x, y, char, name, color, measure,path=None, blocks=False, always_visible = False, fighter=None, ai=None, item=None, equipment = None, rune = None, weapon_level = None):
        self.always_visible = always_visible
        self.x = x
        self.y = y
        self.char = char
        self.name = name        
        self.color = color
        self.measure = measure
        self.path = path
        self.blocks = blocks        
        self.fighter = fighter
        self.equipment = equipment
        self.rune = rune
        self.ai = ai
        self.item = item     
        self.weapon_level = weapon_level


        if self.fighter:  #let the fighter component know who owns it
            self.fighter.owner = self        

        if self.ai:  #let the AI component know who owns it
            self.ai.owner = self        
  
        if self.item:  #let the Item component know who owns it
            self.item.owner = self            
 
        if self.equipment:   #let the Equipment  know who owns it.
            self.equipment.owner = self    
            #there must be an Item component for the Equipment component to work properly
            self.item.owner = self  
            
        if self.rune:   #let the Equipment  know who owns it.
            self.rune.owner = self    
            #there must be an Item component for the Equipment component to work properly
            self.item.owner = self              
            
    def move(self, dx, dy): #Used for the Player's movement 
        #move by the given amount, if the destination is not blocked
        if not is_blocked(self.x + dx, self.y + dy):
            self.x += dx
            self.y += dy          
            
    def move_mob_away(self, dx, dy):
        #move by the given amount, if the destination is not blocked
        
        if not is_blocked(self.x + dx, self.y + dy):
            self.x += dx
            self.y += dy
        else:
            if is_blocked(self.x + dx, self.y + dy):
                if self.x + dx == self.x and self.y +dy == self.y - 1:                                                                                                              ##Heading Up
                    if not is_blocked(self.x -1, self.y + dy) and not is_blocked(self.x +1 , self.y + dy): #To the left and right of O point
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x -= 1
                            self.y += dy
                        else:
                            self.x += 1
                            self.y += dy
                    elif not is_blocked(self.x -1, self.y + dy):    #to the left 
                        self.x -= 1
                        self.y +=dy
                    elif not is_blocked(self.x +1, self.y + dy): #to the right
                        self.x += 1
                        self.y +=dy           
                    
                    elif not is_blocked(self.x -1, self.y) and not is_blocked(self.x +1 , self.y): #to the left and right of start point
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x -= 1
                        else:
                            self.x += 1 
                    elif not is_blocked(self.x -1, self.y): #Left
                        self.x -= 1
                    elif not is_blocked(self.x +1, self.y): #Right
                        self.x += 1
                        
                    elif not is_blocked(self.x - 1, self.y - dy) and not is_blocked(self.x + 1, self.y - dy): #to the left and right of point below
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x -= 1
                            self.y -= dy
                        else:
                            self.x += 1
                            self.y -=dy
                    elif not is_blocked(self.x -1, self.y - dy): #left
                        self.x -= 1
                        self.y - dy
                    elif not is_blocked(self.x +1, self.y - dy): #right
                        self.x += 1
                        self.y -= dy
                        
                    else: 
                        return

                if self.x + dx == self.x + 1 and self.y == self.y + dy:                                                                         ##Heading right  
                    if not is_blocked(self.x + dx, self.y - 1) and not is_blocked(self.x + dx , self.y + 1): 
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x +=dx
                            self.y -= 1
                        else:
                            self.x += dx
                            self.y += 1
                    elif not is_blocked(self.x +dx, self.y - 1):
                        self.x +=dx
                        self.y -=1
                    elif not is_blocked(self.x +dx, self.y + 1):
                        self.x +=dx
                        self.y +=1           
                    
                    elif not is_blocked(self.x, self.y -1) and not is_blocked(self.x , self.y +1):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.y -= 1
                        else:
                            self.y += 1 
                    elif not is_blocked(self.x, self.y -1):
                        self.y -= 1
                    elif not is_blocked(self.x, self.y+1):
                        self.y += 1
                        
                    elif not is_blocked(self.x - dx, self.y - 1) and not is_blocked(self.x - dx , self.y + 1): 
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x -=dx
                            self.y -= 1
                        else:
                            self.x -= dx
                            self.y += 1
                    elif not is_blocked(self.x -dx, self.y - 1):
                        self.x -=dx
                        self.y -=1
                    elif not is_blocked(self.x -dx, self.y + 1):
                        self.x -=dx
                        self.y +=1    
                        
                    else: 
                        return                
                
                if self.x  == self.x + dx and self.y +dy == self.y + 1: #Heading down
                    if not is_blocked(self.x -1, self.y + dy) and not is_blocked(self.x +1 , self.y + dy): #To the left and right of O point
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x -= 1
                            self.y += dy
                        else:
                            self.x += 1
                            self.y += dy
                    elif not is_blocked(self.x -1, self.y + dy):    #to the left 
                        self.x -= 1
                        self.y +=dy
                    elif not is_blocked(self.x +1, self.y + dy): #to the right
                        self.x += 1
                        self.y +=dy           
                    
                    elif not is_blocked(self.x -1, self.y) and not is_blocked(self.x +1 , self.y): #to the left and right of start point
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x -= 1
                        else:
                            self.x += 1 
                    elif not is_blocked(self.x -1, self.y): #Left
                        self.x -= 1
                    elif not is_blocked(self.x +1, self.y): #Right
                        self.x += 1
                        
                    elif not is_blocked(self.x - 1, self.y - dy) and not is_blocked(self.x + 1, self.y - dy): #to the left and right of point below
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x -= 1
                            self.y -= dy
                        else:
                            self.x += 1
                            self.y -=dy
                    elif not is_blocked(self.x -1, self.y - dy): #left
                        self.x -= 1
                        self.y - dy
                    elif not is_blocked(self.x +1, self.y - dy): #right
                        self.x += 1
                        self.y -= dy
                        
                    else: 
                        return                
                
                if self.x + dx == self.x -1 and self.y == self.y + dy: #Heading left
                    if not is_blocked(self.x + dx, self.y - 1) and not is_blocked(self.x + dx , self.y + 1): 
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x +=dx
                            self.y -= 1
                        else:
                            self.x += dx
                            self.y += 1
                    elif not is_blocked(self.x +dx, self.y - 1):
                        self.x +=dx
                        self.y -=1
                    elif not is_blocked(self.x +dx, self.y + 1):
                        self.x +=dx
                        self.y +=1           
                    
                    elif not is_blocked(self.x, self.y -1) and not is_blocked(self.x , self.y +1):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.y -= 1
                        else:
                            self.y += 1 
                    elif not is_blocked(self.x, self.y -1):
                        self.y -= 1
                    elif not is_blocked(self.x, self.y+1):
                        self.y += 1
                        
                    elif not is_blocked(self.x - dx, self.y - 1) and not is_blocked(self.x - dx , self.y + 1): 
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x -=dx
                            self.y -= 1
                        else:
                            self.x -= dx
                            self.y += 1
                    elif not is_blocked(self.x -dx, self.y - 1):
                        self.x -=dx
                        self.y -=1
                    elif not is_blocked(self.x -dx, self.y + 1):
                        self.x -=dx
                        self.y +=1    
                        
                    else: 
                        return 
                
                if self.x + dx == self.x - 1 and self.y + dy == self.y - 1: #UL CHECK dx = -1 and dy = - 1
                    if not is_blocked( self.x, self.y + dy) and not is_blocked( self.x + dx , self.y ):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x += dx
                        else:
                            self.y +=dy
                      
                    elif not is_blocked( self.x, self.y + dy):
                        self.y += dy
                    elif not is_blocked ( self.x + dx, self.y):
                        self.x += dx
                        
                    elif not is_blocked( self.x + dx, self.y - dy) and not is_blocked( self.x - dx , self.y + dy):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x += dx
                            self.y -= dy
                        else:
                            self.x = dx
                            self.y += dy
                            
                    elif not is_blocked(self.x + dx, self.y - dy):
                        self.x += dx
                        self.y -= dy
                    elif not is_blocked( self.x - dx, self.y + dy):
                        self.x -= dx
                        self.y += dy  
                        
                    elif not is_blocked( self.x, self.y - dy) and not is_blocked( self.x - dx, self.y):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0 :
                            self.y -=dy
                        else:
                            self.x -= dx
                            
                    elif not is_blocked( self.x , self.y -dy):
                        self.y -= dy
                    elif not is_blocked( self.x - dx, self.y):
                        self.x -= dx
                        
                    else:
                        return
                        
                if self.x + dx == self.x + 1 and self.y + dy == self.y - 1: #UR CHECK dx = +1 and dy = - 1
                    if not is_blocked( self.x, self.y + dy) and not is_blocked( self.x + dx , self.y ):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x += dx
                        else:
                            self.y +=dy
                      
                    elif not is_blocked( self.x, self.y + dy):
                        self.y += dy
                    elif not is_blocked ( self.x + dx, self.y):
                        self.x += dx
                        
                    elif not is_blocked( self.x + dx, self.y - dy) and not is_blocked( self.x - dx , self.y + dy):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x += dx
                            self.y -= dy
                        else:
                            self.x -= dx
                            self.y += dy
                            
                    elif not is_blocked(self.x + dx, self.y - dy):
                        self.x += dx
                        self.y -= dy
                    elif not is_blocked( self.x - dx, self.y + dy):
                        self.x -= dx
                        self.y += dy  
                        
                    elif not is_blocked( self.x, self.y - dy) and not is_blocked( self.x - dx, self.y):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0 :
                            self.y -=dy
                        else:
                            self.x -= dx
                            
                    elif not is_blocked( self.x , self.y -dy):
                        self.y -= dy
                    elif not is_blocked( self.x - dx, self.y):
                        self.x -= dx
                        
                    else:
                        return        

                if self.x + dx == self.x - 1 and self.y + dy == self.y + 1: #DL CHECK dx = +1 and dy = - 1
                    if not is_blocked( self.x, self.y + dy) and not is_blocked( self.x + dx , self.y ):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x += dx
                        else:
                            self.y +=dy
                      
                    elif not is_blocked( self.x, self.y + dy):
                        self.y += dy
                    elif not is_blocked ( self.x + dx, self.y):
                        self.x += dx
                        
                    elif not is_blocked( self.x + dx, self.y - dy) and not is_blocked( self.x - dx , self.y + dy):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x += dx
                            self.y -= dy
                        else:
                            self.x -= dx
                            self.y += dy
                            
                    elif not is_blocked(self.x + dx, self.y - dy):
                        self.x += dx
                        self.y -= dy
                    elif not is_blocked( self.x - dx, self.y + dy):
                        self.x -= dx
                        self.y += dy  
                        
                    elif not is_blocked( self.x, self.y - dy) and not is_blocked( self.x - dx, self.y):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0 :
                            self.y -=dy
                        else:
                            self.x -= dx
                            
                    elif not is_blocked( self.x , self.y -dy):
                        self.y -= dy
                    elif not is_blocked( self.x - dx, self.y):
                        self.x -= dx
                        
                    else:
                        return             

                if self.x + dx == self.x + 1 and self.y + dy == self.y + 1: #DR CHECK dx = +1 and dy = - 1
                    if not is_blocked( self.x, self.y + dy) and not is_blocked( self.x + dx , self.y ):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x += dx
                        else:
                            self.y +=dy
                      
                    elif not is_blocked( self.x, self.y + dy):
                        self.y += dy
                    elif not is_blocked ( self.x + dx, self.y):
                        self.x += dx
                        
                    elif not is_blocked( self.x + dx, self.y - dy) and not is_blocked( self.x - dx , self.y + dy):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0:
                            self.x += dx
                            self.y -= dy
                        else:
                            self.x -= dx
                            self.y += dy
                            
                    elif not is_blocked(self.x + dx, self.y - dy):
                        self.x += dx
                        self.y -= dy
                    elif not is_blocked( self.x - dx, self.y + dy):
                        self.x -= dx
                        self.y += dy  
                        
                    elif not is_blocked( self.x, self.y - dy) and not is_blocked( self.x - dx, self.y):
                        starai = libtcod.random_get_int(0,0,1)
                        if starai == 0 :
                            self.y -=dy
                        else:
                            self.x -= dx
                            
                    elif not is_blocked( self.x , self.y -dy):
                        self.y -= dy
                    elif not is_blocked( self.x - dx, self.y):
                        self.x -= dx
                        
                    else:
                        return             
            print  str('mondx')+str(dx),  str('mondy')+str(dy)      
                    
                    
    def move_towards(self, target_x, target_y):
        #vector from this object to the target, and distance
        dx = target_x - self.x
        dy = target_y - self.y
        distance = math.sqrt(dx ** 2 + dy ** 2)
 
        #normalize it to length 1 (preserving direction), then round it and
        #convert to integer so the movement is restricted to the map grid
        dx = int(round(dx / distance))
        dy = int(round(dy / distance))
        self.move(dx, dy)
        
    def move_away(self, target_x, target_y):
        dx = target_x - self.x
        dy = target_y - self.y
        
        distance = math.sqrt(dx ** 2 + dy ** 2)
 
        #normalize it to length 1 (preserving direction), then round it and
        #convert to integer so the movement is restricted to the map grid
        dx = int(round(dx / distance))
        dy = int(round(dy / distance))
        self.move_mob_away(-dx, -dy)     
        
    def move_away1(self, target_x, target_y):
        dx = target_x - self.x
        dy = target_y - self.y
        
        distance = math.sqrt(dx ** 2 + dy ** 2)
 
        #normalize it to length 1 (preserving direction), then round it and
        #convert to integer so the movement is restricted to the map grid
        dx = int(round(dx / distance))
        dy = int(round(dy / distance))
        self.move_mob_away(-dx, -dy)               

    def distance_to(self, other):
        #return the distance to another object
        dx = other.x - self.x
        dy = other.y - self.y
        return math.sqrt(dx ** 2 + dy ** 2)        
        
    def distance(self, x, y):
        #return the distance to some coordinates
        return math.sqrt((x - self.x) ** 2 + (y - self.y) ** 2)        
        
    def send_to_back(self):
        #make this object be drawn first, so all others appear above it if they're in the same tile.
        global objects
        objects.remove(self)
        objects.insert(0, self)
        
    def draw(self):
        #only show if it's visible to the player
        if libtcod.map_is_in_fov(fov_map, self.x, self.y) or (self.always_visible and map[self.x][self.y].explored):
            #set the color and then draw the character that represents this object at its position
            libtcod.console_set_default_foreground(con, self.color)
            libtcod.console_put_char(con, self.x, self.y, self.char, libtcod.BKGND_NONE)        
 
    def clear(self):
        #erase the character that represents this object
        libtcod.console_put_char(con, self.x, self.y, ' ', libtcod.BKGND_NONE)    
    
## Object>>Fighter ##                
class Fighter:
    
    #combat-related properties and methods (monster, player, NPC).
    def __init__(self,type, hp, defence, power, xp,luck, death_function=None):
        self.base_max_hp = hp
        self.hp = hp
        self.base_defence = defence
        self.base_power = power
        self.xp = xp
        self.base_luck = luck
        self.type = type
        self.death_function = death_function
        
    @property
    def power(self):  
        bonus = sum(equipment.power_bonus for equipment in get_all_equipped(self.owner))  
        return self.base_power + bonus
                        
    @property
    def rune_power(self):           
        effects = 2
        bonus = sum(equipment.power_bonus for equipment in get_all_equipped(self.owner))    
        message ( name_fetch.got_name() + ' gets help from the ' + str(player.fighter.type) + ' blessing' , libtcod.orange)
        return (self.base_power * effects) + bonus       
        
    @property
    def luck(self):
        bonus = sum(equipment.luck_bonus for equipment in get_all_equipped(self.owner))
        return self.base_luck + bonus
          
    @property
    def defence(self):  #return actual defence, by summing up the bonuses from all equipped items
        bonus = sum(equipment.defence_bonus for equipment in get_all_equipped(self.owner))
        return self.base_defence + bonus
 
    @property
    def max_hp(self):  #return actual max_hp, by summing up the bonuses from all equipped items
        bonus = sum(equipment.max_hp_bonus for equipment in get_all_equipped(self.owner))
        return self.base_max_hp + bonus
       
    def attack(self, target):     
        global display_damage
    
        strike_luck = libtcod.random_get_int(0,0,100)
        if strike_luck <= self.luck:
            damage = self.power*2 - target.fighter.defence
            
        elif target.fighter.type == str('box'):
            damage = 1
                    
        elif target.fighter.type == str('Beastmob') and player.fighter.type == str('Dwarven'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Drakemob') and player.fighter.type == str('Dwarven'):
            damage = self.rune_power - target.fighter.defence
            
        elif target.fighter.type == str('Manmob') and player.fighter.type == str('Dwarven'):
            damage = self.rune_power - target.fighter.defence 
            

        elif target.fighter.type == str('Beastmob') and player.fighter.type == str('Elven'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Corruptmob') and player.fighter.type == str('Elven'):
            damage = self.rune_power - target.fighter.defence          

        elif target.fighter.type == str('Manmob') and player.fighter.type == str('Elven'):
            damage = self.rune_power - target.fighter.defence
            

        elif target.fighter.type == str('Beastmob') and player.fighter.type == str('Beasts'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Undeadmob') and player.fighter.type == str('Beasts'):
            damage = self.rune_power - target.fighter.defence

            
        elif target.fighter.type == str('Dwarvenmob') and player.fighter.type == str('Drake'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Corruptmob') and player.fighter.type == str('Drake'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Manmob') and player.fighter.type == str('Drake'):
            damage = self.rune_power - target.fighter.defence
            

        elif target.fighter.type == str('Beastmob') and player.fighter.type == str('Corrupt'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Manmob') and player.fighter.type == str('Corrupt'):
            damage = self.rune_power - target.fighter.defence
            

        elif target.fighter.type == str('Elvenmob') and player.fighter.type == str('Undead'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Beastmob') and player.fighter.type == str('Undead'):
            damage = self.rune_power - target.fighter.defence     

        elif target.fighter.type == str('Dwarvenmob') and player.fighter.type == str('Undead'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Corruptmob') and player.fighter.type == str('Undead'):
            damage = self.rune_power - target.fighter.defence
            

        elif target.fighter.type == str('Elvenmob') and player.fighter.type == str('Evil'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Beastmob') and player.fighter.type == str('Evil'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Corruptmob') and player.fighter.type == str('Evil'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Manmob') and player.fighter.type == str('Evil'):
            damage = self.rune_power - target.fighter.defence     
            

        elif target.fighter.type == str('Elvenmob') and player.fighter.type == str('Valar'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Corruptmob') and player.fighter.type == str('Valar'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Undeadmob') and player.fighter.type == str('Valar'):
            damage = self.rune_power - target.fighter.defence

        elif target.fighter.type == str('Evilmob') and player.fighter.type == str('Valar'):
            damage = self.rune_power - target.fighter.defence            
            
            
        elif target.fighter.type == str('Beastmob') and player.fighter.type == str('Men'):
            damage = self.rune_power - target.fighter.defence            
            
        elif target.fighter.type == str('Evilmob') and player.fighter.type == str('Men'):
            damage = self.rune_power - target.fighter.defence     

        elif target.fighter.type == str('Evilmob'):
            damage = 0    
            
        else:
            damage = self.power - target.fighter.defence  ## WHERE THE MAGIC HAPPENS
  
        if damage > 0:            
             
            target.fighter.take_damage(damage)
            if self.owner != player:
                message (self.owner.name.capitalize() + ' deals ' + str(damage) + ' damage.', libtcod.white )        
            else:
                display_damage = damage
                
        else:
            message (self.owner.name.capitalize() + "'s attack does nothing", libtcod.white)      
        
    def take_damage(self, damage):
        global record_xp 
        global display , display_max_hp
        #apply damage if possible
        if damage > 0:
            self.hp -= damage 
            if self.owner != player:
                display = self.hp
                display_max_hp = self.base_max_hp                
            #check for death. if there's a death function, call it
            if self.hp <= 0:
                function = self.death_function
                if function is not None:
                    function(self.owner)            
                    
                if self.owner != player:  #yield experience to the player
                    #USE THIS TO DECREASE MOB EXP YEILD!!
                    player.fighter.xp += self.xp
                    record_xp += self.xp
                    wpn = self.xp 
                    for item in inventory:
                    #show additional information, in case it's equipped
                        if item.equipment and item.equipment.is_equipped:
                            if item.weapon_level:
                                item.weapon_level.gain_xp(wpn)
                                
                                if item.weapon_level.is_id == False and item.weapon_level.wpn_level == 10:
                                    item.weapon_level.is_id = True
                                    message( name_fetch.got_name() + ' has identified the ' + str(item.weapon_level.id_name) + '.' , libtcod.green)   
                                    function = item.weapon_level.id_function
                                    if function is not None:
                                        function(item)  
                    
    def heal(self, amount):
        #heal by the given amount, without going over the maximum
        self.hp += amount
        if self.hp > self.max_hp:
            self.hp = self.max_hp
                        

## Object>>Fighter>>BasicMob ## 
class StunnedMonster:
    #AI for a temporarily confused monster (reverts to previous AI after a while).
    def __init__(self, old_ai, num_turns=STUN_NUM_TURNS):
        self.old_ai = old_ai
        self.num_turns = num_turns
 
    def take_turn(self):
        if self.num_turns > 0:  #still confused...
            #move in a random direction, and decrease the number of turns confused
            self.owner.move(0,0)
            self.num_turns -= 1
 
        else:  #restore the previous AI (this one will be deleted because it's not referenced anymore)
            self.owner.ai = self.old_ai
            message('The ' + self.owner.name + ' is no longer stunned!', libtcod.red)
                
class NotHostileMonster:
    #AI for a basic monster.
    def take_turn(self):
        #a basic monster takes its turn. If you can see it, it can see you
        monster = self.owner
        
        if monster.distance_to(player) <= 4:
            monster.move_away(player.x, player.y)
     #   if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):
           # print str('monx')+str(monster.x) ,  str('mony')+str(monster.y) ,  str('mondistance')+str(monster.distance_to(player))
        else:
                    #our path is empty, just wander randomly
            randdirection = libtcod.random_get_int(0,1,8)
            if randdirection == 1:
                monster.move(-1,1)
            elif randdirection == 2:
                monster.move(0,1)
            elif randdirection == 3:
                monster.move(1,1)
            elif randdirection == 4:
                monster.move(-1,0)
            elif randdirection == 5:
                monster.move(1,0)
            elif randdirection == 6:
                monster.move(-1,-1)
            elif randdirection == 7:
                monster.move(0,-1)
            else:
                monster.move(1,-1)              
                
class WanderAI:
    #AI for a basic monster
    def take_turn(self):
        #a basic monster takes its turn. If you can see it, it can see you.
        monster = self.owner

        #first, check its path item exists
   
        if monster.path == None:
            monster.path = libtcod.path_new_using_map(fov_map,1.41)
        if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):      
            #move towards the player if far away
            if monster.distance_to(player) >= 2:                                             
                #compute how to reach the player
                libtcod.path_compute(monster.path,monster.x, monster.y, player.x, player.y)
                #and move one square towards them
                nextx,nexty=libtcod.path_walk(monster.path,True)
                monster.move_towards(nextx,nexty)
                print monster.distance_to(player)
            #now we're close enough to attack! ...assuming the player is alive
            elif player.fighter.hp > 0:
                monster.fighter.attack(player)
        else:
                #the player cannot see the monster
                #if we have an old path, continue to follow it -
                # this means that monsters you dodge will go to where they last saw you!
            if not libtcod.path_is_empty(monster.path):
                nextx,nexty=libtcod.path_walk(monster.path,True)
                monster.move_towards(nextx,nexty)
            else:
                    #our path is empty, just wander randomly
                randdirection = libtcod.random_get_int(0,1,8)
                if randdirection == 1:
                    monster.move(-1,1)
                elif randdirection == 2:
                    monster.move(0,1)
                elif randdirection == 3:
                    monster.move(1,1)
                elif randdirection == 4:
                    monster.move(-1,0)
                elif randdirection == 5:
                    monster.move(1,0)
                elif randdirection == 6:
                    monster.move(-1,-1)
                elif randdirection == 7:
                    monster.move(0,-1)
                else:
                    monster.move(1,-1)                        
        
class init_SleepingAI:
    def __init__(self):
        self.state = 'sleeping'
 
    def take_turn(self):
        monster = self.owner
        
        if monster.fighter.hp <= monster.fighter.base_max_hp/3:
            self.state = 'flee'   
            
        if monster.path == None:
            monster.path = libtcod.path_new_using_map(fov_map,1.41)    
    
        if self.state == 'sleeping': 
            if monster.distance_to(player) < 3:
                wakeup_random = libtcod.random_get_int(0, 0 , 100)
                if wakeup_random < 60:
                    self.state = 'chasing'
                    message(monster.name + ' has woken up!', libtcod.red)
                else:
                    message(monster.name + ' is still sleeping!', libtcod.green)
                    player.fighter.xp += monster.fighter.xp/5
            if monster.distance_to(player) < 2:
                self.state = 'chasing'
                message(monster.name + ' has woken up!', libtcod.red)
                
        elif self.state == 'chasing':     
            if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):      
                if monster.distance_to(player) >= 2:                                             
                    libtcod.path_compute(monster.path,monster.x, monster.y, player.x, player.y)
                    nextx,nexty=libtcod.path_walk(monster.path,True)
                    monster.move_towards(nextx,nexty)
                    print monster.distance_to(player)
                elif player.fighter.hp > 0:
                    monster.fighter.attack(player)       
            else:
                self.state = 'wondering'
            
        elif self.state == 'wondering':
            if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):   
                if monster.distance_to(player) >= 2:                                             
                    libtcod.path_compute(monster.path,monster.x, monster.y, player.x, player.y)
                    nextx,nexty=libtcod.path_walk(monster.path,True)
                    monster.move_towards(nextx,nexty)            
                self.state = 'chasing'
                return
        
            if not libtcod.path_is_empty(monster.path):
                nextx,nexty=libtcod.path_walk(monster.path,True)
                monster.move_towards(nextx,nexty)
            else:
                    #our path is empty, just wander randomly
                randdirection = libtcod.random_get_int(0,1,8)
                if randdirection == 1:
                    monster.move(-1,1)
                elif randdirection == 2:
                    monster.move(0,1)
                elif randdirection == 3:
                    monster.move(1,1)
                elif randdirection == 4:
                    monster.move(-1,0)
                elif randdirection == 5:
                    monster.move(1,0)
                elif randdirection == 6:
                    monster.move(-1,-1)
                elif randdirection == 7:
                    monster.move(0,-1)
                else:
                    monster.move(1,-1)   
                
        elif self.state == 'flee':
            monster.move_away(player.x, player.y)
            
class PatrolAI:
    def __init__(self):
        self.state = 'wondering'
 
    def take_turn(self):
        monster = self.owner
        
        if monster.fighter.hp <= monster.fighter.base_max_hp/3:
            self.state = 'flee'   
            
        if monster.path == None:
            monster.path = libtcod.path_new_using_map(fov_map,1.41)    
                
        elif self.state == 'chasing':     
            if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):      
                if monster.distance_to(player) >= 2:                                             
                    libtcod.path_compute(monster.path,monster.x, monster.y, player.x, player.y)
                    nextx,nexty=libtcod.path_walk(monster.path,True)
                    monster.move_towards(nextx,nexty)
                    print monster.distance_to(player)
                elif player.fighter.hp > 0:
                    monster.fighter.attack(player)       
            else:
                self.state = 'wondering'
            
        elif self.state == 'wondering':
            if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):   
                if monster.distance_to(player) >= 2:                                             
                    libtcod.path_compute(monster.path,monster.x, monster.y, player.x, player.y)
                    nextx,nexty=libtcod.path_walk(monster.path,True)
                    monster.move_towards(nextx,nexty)            
                self.state = 'chasing'
                return
        
            if not libtcod.path_is_empty(monster.path):
                nextx,nexty=libtcod.path_walk(monster.path,True)
                monster.move_towards(nextx,nexty)
            else:
                    #our path is empty, just wander randomly
                randdirection = libtcod.random_get_int(0,1,8)
                if randdirection == 1:
                    monster.move(-1,1)
                elif randdirection == 2:
                    monster.move(0,1)
                elif randdirection == 3:
                    monster.move(1,1)
                elif randdirection == 4:
                    monster.move(-1,0)
                elif randdirection == 5:
                    monster.move(1,0)
                elif randdirection == 6:
                    monster.move(-1,-1)
                elif randdirection == 7:
                    monster.move(0,-1)
                else:
                    monster.move(1,-1)   
                
        elif self.state == 'flee':
            monster.move_away(player.x, player.y)            
            
class Not_Hostile:
    def take_turn(self):
    
        monster = self.owner
        if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):
    
            if monster.distance_to(player) < 5:
                monster.move_away(player.x,player.y)
                        
        
## ITEMS ##                
class Item:
    #an item that can be picked up and used.
    def __init__(self, weight, use_function=None):
        self.use_function = use_function
        self.weight = weight
 
    def pick_up(self):
        global weight_inventory
        
        #add to the player's inventory and remove from the map
        if len(inventory) >= 26:
            message(name_fetch.got_name() + "'s inventory is full and can not carry more " + self.owner.name + "s.", libtcod.red)
            
        elif weight_inventory + self.weight <= weight_cap: 
            inventory.append(self.owner)
            objects.remove(self.owner)
            weight_inventory += self.weight
            
            message(name_fetch.got_name() + ' picked up a ' + self.owner.name + '!', libtcod.green)            
        else:
            message(name_fetch.got_name() + "'s inventory is full and can not carry more " + self.owner.name + "s.", libtcod.red)   
            
    def drop(self):
        global weight_inventory
        #special case: if the object has the Equipment component, dequip it before dropping
        if self.owner.equipment:
            self.owner.equipment.dequip()    
    
        #add to the map and remove from the player's inventory. also, place it at the player's coordinates
        objects.append(self.owner)
        inventory.remove(self.owner)
        self.owner.x = player.x
        self.owner.y = player.y
        weight_inventory -= self.weight
        message(name_fetch.got_name() + ' drops the ' + self.owner.name + '.', libtcod.yellow)    
        message( str(weight_inventory), libtcod.white)
 
    def use(self):
        global weight_inventory
        if self.owner.equipment:
            self.owner.equipment.toggle_equip()
            return    
        elif self.owner.rune:
            self.owner.rune.toggle_enchant()
            weight_inventory -= self.weight
            inventory.remove(self.owner)
            return
    
        #just call the "use_function" if it is defined
        if self.use_function is None:
            message('The ' + self.owner.name + ' cannot be used.')
        else:
            if self.use_function() != 'cancelled':
                weight_inventory -= self.weight
                inventory.remove(self.owner)  #destroy after use, unless it was cancelled for some reason
                
        #special case: if the object has the Equipment component, the "use" action is to equip/dequip
               
                
## Object Blocking ##  Move above create_room #work
def is_blocked(x, y):
    #first test the map tile
    if map[x][y].blocked:
        return True
 
    #now check for any blocking objects
    for object in objects:
        if object.blocks and object.x == x and object.y == y:
            return True
 
    return False    
    
class rune:

    def __init__(self,slot,type):
        self.type = type
        self.slot = slot
        self.is_enchanted = False
            
 
    def toggle_enchant(self):  #toggle equip/dequip status
        if self.is_enchanted:
            self.dechant()
        else:
            self.enchant()
 
    def enchant(self):
        #if the slot is already being used, dequip whatever is there first
        old_enchantment = get_enchanted_in_slot(self.slot)
        if old_enchantment is not None:
            old_enchantment.dechant()    
    
        #equip object and show a message about it
        self.is_enchanted = True
        player.fighter.type = self.type
        message( str(name_fetch.got_name()) + ' is blessed by the rune!', libtcod.light_green)
 
    def dechant(self):
        #dequip object and show a message about it
        if not self.is_enchanted: return
        self.is_enchanted = False
        message('Dequipped ' + self.owner.name + ' from ' + self.slot + '.', libtcod.light_yellow)
        
def get_enchanted_in_slot(slot):  #returns the equipment in a slot, or None if it's empty
    for obj in inventory:
        if obj.rune and obj.rune.slot == slot and obj.rune.is_enchanted:
            return obj.rune
    return None        
    
def get_all_enchantments(obj):  #returns a list of equipped items
    if obj == player:
        enchant_list = []
        for item in inventory:
            if item.rune and item.rune.is_enchanted:
                enchant_list.append(item.rune)
        return enchant_list
    else:
        return []  #other objects have no equipment  
            
### CLASS EQUIPMENT ###    

class Equipment:
    #an object that can be equipped, yielding bonuses. automatically adds the Item component. 
    def __init__(self, slot, power_bonus=0, defence_bonus=0, max_hp_bonus=0, luck_bonus=0 , light_bonus = 0 , step_lives = 0):
        self.power_bonus = power_bonus
        self.defence_bonus = defence_bonus
        self.max_hp_bonus = max_hp_bonus
        self.luck_bonus = luck_bonus
        self.light_bonus = light_bonus
        self.step_lives = step_lives
        self.slot = slot
        self.is_equipped = False
        
             
    def toggle_equip(self):  #toggle equip/dequip status
        if self.is_equipped:
            self.dequip()
        else:
            self.equip()
 
    def equip(self):
        #if the slot is already being used, dequip whatever is there first
        old_equipment = get_equipped_in_slot(self.slot)
        if old_equipment is not None:
            old_equipment.dequip()    
    
        #equip object and show a message about it
        self.is_equipped = True
        message('Equipped ' + self.owner.name + ' on ' + self.slot + '.', libtcod.light_green)
 
    def dequip(self):
        #dequip object and show a message about it
        if not self.is_equipped: return
        self.is_equipped = False
        message('Dequipped ' + self.owner.name + ' from ' + self.slot + '.', libtcod.light_yellow)
                
class Weapon_Level:

    def __init__ (self, id_name, xp_multiplier, xp_base, xp_top, wpn_bonus, wpn_level, id_function = None ):
        self.id_name = id_name
        self.xp_multiplier = xp_multiplier
        self.xp_base = xp_base
        self.xp_top = xp_top
        self.wpn_bonus = wpn_bonus
        self.wpn_level = wpn_level
        self.id_function = id_function
        self.is_id = False
                
    def gain_xp(self, wpn):
        catch = wpn*self.xp_multiplier
        if self.wpn_level < 10:
            self.xp_base += catch 
            if self.xp_base > self.xp_top:
                self.xp_base -= self.xp_top
                self.wpn_level += 1
                if self.xp_base > self.xp_top:
                    self.xp_base -= self.xp_top
                    self.wpn_level += 1

            
            
        print self.xp_base , self.wpn_level , self.is_id
                    
def get_equipped_in_slot(slot):  #returns the equipment in a slot, or None if it's empty
    for obj in inventory:
        if obj.equipment and obj.equipment.slot == slot and obj.equipment.is_equipped:
            return obj.equipment
    return None        
    
def get_all_equipped(obj):  #returns a list of equipped items
    if obj == player:
        equipped_list = []
        for item in inventory:
            if item.equipment and item.equipment.is_equipped:
                equipped_list.append(item.equipment)
        return equipped_list
    else:
        return []  #other objects have no equipment    
    
###############
#DungeonPieces#
###############
def create_room(room):
    global map
    #go through the tiles in the rectangle and make them passable
    for x in range(room.x1 + 1, room.x2):
        for y in range(room.y1 + 1, room.y2):
            map[x][y].blocked = False
            map[x][y].block_sight = False
            
#Horizontal Tunnels            
def create_h_tunnel(x1, x2, y):
    global map
    for x in range(min(x1, x2), max(x1, x2) + 1):
        map[x][y].blocked = False
        map[x][y].block_sight = False
        
#Vertical Tunnels
def create_v_tunnel(y1, y2, x):
    global map
    #vertical tunnel
    for y in range(min(y1, y2), max(y1, y2) + 1):
        map[x][y].blocked = False
        map[x][y].block_sight = False
        

############
#MAP Looper#
############
def make_map():
    global map, objects
    global RAN_Structure
    global stairs
    
    #the list of objects starting with the player
    objects = [player]
 
    #fill map with "blocked" tiles
    map = [[ Tile(True)
        for y in range(MAP_HEIGHT) ]
            for x in range(MAP_WIDTH) ]
 
    rooms = []
    num_rooms = 0
 
    for r in range(MAX_ROOMS):
        #random width and height
        w = libtcod.random_get_int(0, ROOM_MIN_SIZE, ROOM_MAX_SIZE)
        h = libtcod.random_get_int(0, ROOM_MIN_SIZE, ROOM_MAX_SIZE)
        #random position without going out of the boundaries of the map
        x = libtcod.random_get_int(0, 0,MAP_WIDTH - w - 1) 
        y = libtcod.random_get_int(0, 0, MAP_HEIGHT - h - 1)
        # Random Room Structure Generator #
        RAN_Structure =  libtcod.random_get_int(0,1,6)
 
        #"Rect" class makes rectangles easier to work with
        new_room = Rect(x, y, w, h)
 
        #run through the other rooms and see if they intersect with this one
        failed = False
        for other_room in rooms:
            if new_room.intersect(other_room):
                failed = True
                break
                              
        if not failed:
            #this means there are no intersections, so this room is valid
 
            #"paint" it to the map's tiles
            create_room(new_room)
              
            #center coordinates of new room, will be useful later
            (new_x, new_y) = new_room.center()
 
            if (3 < w) or (3  < h):
                if RAN_Structure == int(1):
                    map[new_x-1][new_y-1].blocked = True
                    map[new_x-1][new_y-1].block_sight = True
                    map[new_x-1][new_y-1].is_blocked =True
                    
                    map[new_x+1][new_y-1].blocked = True
                    map[new_x+1][new_y-1].block_sight = True
                    map[new_x+1][new_y-1].is_blocked =True                    
                    
                    map[new_x-1][new_y+1].blocked = True
                    map[new_x-1][new_y+1].block_sight = True
                    map[new_x-1][new_y+1].is_blocked =True
                    
                    map[new_x+1][new_y+1].blocked = True
                    map[new_x+1][new_y+1].block_sight = True      
                    map[new_x+1][new_y+1].is_blocked =True
 
             #add some contents to this room, such as monsters
            place_objects(new_room)
 
            if num_rooms == 0:
                #this is the first room, where the player starts at
                player.x = new_x
                player.y = new_y
            else:
                #all rooms after the first:
                #connect it to the previous room with a tunnel
 
                #center coordinates of previous room
                (prev_x, prev_y) = rooms[num_rooms-1].center()
 
                #draw a coin (random number that is either 0 or 1)
                if libtcod.random_get_int(0, 0, 1) == 1:
                    #first move horizontally, then vertically
                    create_h_tunnel(prev_x, new_x, prev_y)
                    create_v_tunnel(prev_y, new_y, new_x)
                else:
                    #first move vertically, then horizontally
                    create_v_tunnel(prev_y, new_y, prev_x)
                    create_h_tunnel(prev_x, new_x, new_y)
 
            #finally, append the new room to the list
            rooms.append(new_room)
            num_rooms += 1
            
    #create stairs at the center of the last room
    stairs = Object(new_x, new_y, '<', 'stairs', libtcod.white,0, always_visible = True)
    objects.append(stairs)
    stairs.send_to_back()  #so it's drawn below the monsters
    
def random_choice_index(chances):  #choose one option from list of chances, returning its index
    #the dice will land on some number between 1 and the sum of the chances
    dice = libtcod.random_get_int(0, 1, sum(chances))
 
    #go through all chances, keeping the sum so far
    running_sum = 0
    choice = 0
    for w in chances:
        running_sum += w
 
        #see if the dice landed in the part that corresponds to this choice
        if dice <= running_sum:
            return choice
        choice += 1
                    
                    
def random_choice(chances_dict):
    #choose one option from dictionary of chances, returning its key
    chances = chances_dict.values()
    strings = chances_dict.keys()
 
    return strings[random_choice_index(chances)]
    
def from_dungeon_level(table):
    #returns a value that depends on level. the table specifies what value occurs after each level, default is 0.
    for (value, level) in reversed(table):
        if dungeon_level >= level:
            return value
    return 0
    
################                    
#Object Placement#                    
################
def place_objects(room):
    global shock_bonus
    #this is where we decide the chance of each monster or item appearing.
 
    #maximum number of monsters per room
    max_monsters = from_dungeon_level([[2, 1],[4,2] , [3,3],[4, 4], [4, 8], [ 5 ,11]]) #2 = max monsters, 1 =  floor level.
 
    #chance of each monster
    monster_chances = {}
    monster_chances['chest'] = from_dungeon_level([[5,1]])
    monster_chances['tomb'] = from_dungeon_level([[5,1]])
    
    monster_chances['rabbit'] = from_dungeon_level ([[60,1],[10,2],[0,3]])                                                                                                      #-2
    
    monster_chances['warg pup'] = from_dungeon_level([[ 70, 2], [30,3], [ 0 , 4]])                                                                                                           # 2  
    monster_chances['skeletonnewb'] = from_dungeon_level([[20, 2], [60 , 3], [ 50 , 4] , [ 0 , 5]])                                                                                            # 2
    
    monster_chances['crebain'] = from_dungeon_level([[ 40 , 3 ],[ 50, 4], [5 , 5], [ 0 , 6]])                                                                                            # 3    
    
    monster_chances['skeleton'] = from_dungeon_level([[10, 4], [ 50, 5], [0, 6]])                                                                                          # 4    
    monster_chances['wolf'] = from_dungeon_level([[20, 3], [40,5], [0,7]])                                                                                                                         # 4    
    monster_chances['mewlip'] = from_dungeon_level([[5, 4], [ 10, 5], [50, 7], [0,8]])                                                                                                                       # 4    
    
    monster_chances['spectrenewb'] = from_dungeon_level([[10, 5], [20, 6], [50, 7] , [ 0,9]])                                                                                   # 5    
    
    monster_chances['greywolf'] = from_dungeon_level([[40, 6], [0, 10]])                                                                                                                  # 6    
    
    monster_chances['snaga'] = from_dungeon_level([[40, 7], [0,11]])                                                                                                                        #8        
    monster_chances['warg'] = from_dungeon_level([[30, 8], [60, 9]]) # 15 = chance of showing 3 = floor level                                       # 8

    
    monster_chances['spectre'] = from_dungeon_level([[20, 9]])                                                                                                                      #10
    monster_chances['snuffler'] = from_dungeon_level([[60, 10]])                                                                                                                       #10    
    monster_chances['wargleader'] = from_dungeon_level([[40, 10]])                                                                                                                   #10

    
    max_items = from_dungeon_level([[3, 1]])

    item_chances = {}
    item_chances['lembas'] = from_dungeon_level([[40, 1], [0,2]])
    item_chances['Throwing Axe'] = 20
    item_chances['scroll of stun'] = 0

    item_chances['DwarvenRune'] = from_dungeon_level([[5, 3], [0,6]]) 
    item_chances['ElvenRune'] = from_dungeon_level([[5, 3], [0,6]])      
    item_chances['BeastRune'] = from_dungeon_level([[5, 1], [0,2]])  
    item_chances['DrakeRune'] = 0    
    item_chances['CorruptRune'] = from_dungeon_level([[5, 3], [0,6]])   
    item_chances['UndeadRune'] =from_dungeon_level([[5, 3], [0,6]])   
    item_chances['EvilRune'] = 0
    item_chances['ValarRune'] = 0
    item_chances['ManRune'] = 0
    
    item_chances['sword'] =     from_dungeon_level([[5, 1]])
    item_chances['dwarvenhammer'] = from_dungeon_level([[40,1]])
    item_chances['woodcuttersaxe'] = from_dungeon_level([[5,1]])
    item_chances['orcmace'] = from_dungeon_level ([[5,1]])
    
    item_chances['shield'] =    from_dungeon_level([[20, 2]])    
    item_chances['helm'] = from_dungeon_level([[20, 2]])
    item_chances['steelplates'] = from_dungeon_level([[20,2]])
    item_chances['gloves'] = from_dungeon_level([[820,2]])
    item_chances['steelleggings'] = from_dungeon_level([[20, 2]])
    item_chances['boots'] = from_dungeon_level([[20,2]])
    
    item_chances['amethyst'] = from_dungeon_level([[10,2]])    
    
    item_chances['Lamp of Ethu'] = 30    
    
    #choose random number of monsters
    num_monsters = libtcod.random_get_int(0, 0, max_monsters)
 
    for i in range(num_monsters):
        #choose random spot for this monster
        x = libtcod.random_get_int(0, room.x1+1, room.x2-1)
        y = libtcod.random_get_int(0, room.y1+1, room.y2-1)
 
         #only place it if the tile is not blocked
        if not is_blocked(x, y):
            choice = random_choice(monster_chances)
            
            if choice == 'rabbit':
            
                fighter_component = Fighter('Beastmob',hp=1, defence=0, power=0, xp = 100, luck = 0, death_function=beast_death)
                #ai_component = NotHostileMonster()
                #ai_component = WanderAI()          
                ai_component = PatrolAI()
                monster = Object(x, y, int(195), "rabbit", libtcod.light_blue,0, blocks=True, fighter=fighter_component, ai=ai_component)      

            elif choice == 'chest':
            
                fighter_component = Fighter('box', hp=2, defence=0,power=0,xp=20,luck=0, death_function=box_death)
                monster = Object(x,y, int(224), 'chest', libtcod.light_azure,0,blocks=True, fighter=fighter_component, always_visible= True)
                
            elif choice == 'tomb':
            
                fighter_component = Fighter('box', hp=2, defence=0,power=0,xp=20,luck=0, death_function=box_death)
                monster = Object(x,y, int(224), 'tomb', libtcod.light_azure,0,blocks=True, fighter=fighter_component, always_visible= True)                
                
                
                
            
            elif choice == 'warg pup': 
            
                fighter_component = Fighter("Corruptmob",hp=2, defence=0, power=3, xp = 25, luck = 1, death_function=corrupt_death)
                ai_component = WanderAI() 
                monster = Object(x, y, int(193), "warg pup", libtcod.amber,0, blocks=True, fighter=fighter_component, ai=ai_component)
                
            elif choice == 'skeletonnewb':
          
                fighter_component = Fighter("Undeadmob",hp=4, defence=1, power=5, xp = 50, luck = 2, death_function= undead_death)
                ai_component = WanderAI()
                monster = Object(x, y, 's', "skeleton", libtcod.white,0, blocks=True, fighter=fighter_component, ai=ai_component)               
                
            elif choice == 'crebain':
          
                fighter_component = Fighter("Beastmob",hp=4, defence=0, power=5, xp = 50, luck = 2, death_function= beast_death)
                ai_component = WanderAI()
                monster = Object(x, y, 'm', "crebain", libtcod.white,0, blocks=True, fighter=fighter_component, ai=ai_component)      
                
            elif choice == 'wolf':
          
                fighter_component = Fighter("Beastmob",hp=6, defence=2, power=5, xp = 60, luck = 1, death_function= beast_death)
                ai_component = WanderAI()
                monster = Object(x, y, 'w', "wolf", libtcod.light_sepia,0, blocks=True, fighter=fighter_component, ai=ai_component)                     
                
            elif choice == 'mewlip':
          
                fighter_component = Fighter("Corruptmob",hp=10, defence=4, power=6, xp = 100, luck = 5, death_function= corrupt_death)
                ai_component = WanderAI()
                monster = Object(x, y, 'P', "mewlip", libtcod.celadon,0, blocks=True, fighter=fighter_component, ai=ai_component)                      
                
            elif choice == 'skeleton':
          
                fighter_component = Fighter("Undeadmob",hp=10, defence=0, power=7, xp = 80, luck = 2, death_function= undead_death)
                ai_component = WanderAI()
                monster = Object(x, y, 'S', "skeleton soldier", libtcod.silver, 0,blocks=True, fighter=fighter_component, ai=ai_component)                      
                
            elif choice == 'spectrenewb':
          
                fighter_component = Fighter("Evilmob",hp=5, defence=2, power=7, xp = 120, luck = 2, death_function= evil_death)
                ai_component = WanderAI()
                monster = Object(x, y, 'e', "spectre", libtcod.lighter_lime,0, blocks=True, fighter=fighter_component, ai=ai_component)                     
                
            elif choice == 'greywolf':
          
                fighter_component = Fighter("Beastmob",hp=12, defence=4, power=8, xp = 140, luck = 2, death_function= beast_death)
                ai_component = WanderAI()
                monster = Object(x, y, 'w', "greywolf", libtcod.lightest_grey,0, blocks=True, fighter=fighter_component, ai=ai_component)       
 
            elif choice == 'snaga':
          
                fighter_component = Fighter("Corruptmob",hp=10, defence=6, power=8, xp = 150, luck = 1, death_function= corrupt_death)
                ai_component = WanderAI()
                monster = Object(x, y, 'o', "snaga", libtcod.dark_chartreuse, 0,blocks=True, fighter=fighter_component, ai=ai_component)      
 
            elif choice == 'spectre':
          
                fighter_component = Fighter("Evilmob",hp=10, defence=4, power=12, xp = 200, luck = 5, death_function= evil_death)
                ai_component = WanderAI()
                monster = Object(x, y, 'E', "royal spectre", libtcod.white,0, blocks=True, fighter=fighter_component, ai=ai_component)       
 
            elif choice == 'snuffler':
          
                fighter_component = Fighter("Corruptmob",hp=14, defence=5, power=12, xp = 210, luck = 1, death_function= corrupt_death)
                ai_component = WanderAI()
                monster = Object(x, y, 'o', "snuffler", libtcod.dark_green,0, blocks=True, fighter=fighter_component, ai=ai_component)      

            elif choice == 'wargleader':
          
                fighter_component = Fighter("Corruptmob",hp=17, defence=0, power=14, xp = 230, luck = 2, death_function= corrupt_death)
                ai_component = WanderAI()
                monster = Object(x, y, int(193), "warg leader", libtcod.silver, 0,blocks=True, fighter=fighter_component, ai=ai_component)      





                
 
            elif choice == 'warg':
            
                #create a warg
                fighter_component = Fighter("Corruptmob",hp=15, defence=0, power=10, xp = 170, luck = 2, death_function=corrupt_death)
                ai_component = BasicMonster()
                monster = Object(x, y, int(193), "warg", libtcod.dark_orange,0, blocks=True, fighter=fighter_component, ai=ai_component)
 
            objects.append(monster)         
            
    #choose random number of items
    num_items = libtcod.random_get_int(0, 0, max_items)
 
    for i in range(num_items):
        #choose random spot for this item
        x = libtcod.random_get_int(0, room.x1+1, room.x2-1)
        y = libtcod.random_get_int(0, room.y1+1, room.y2-1)
 
        #only place it if the tile is not blocked
        if not is_blocked(x, y):
            choice = random_choice(item_chances)
            
            
            
            if choice == 'lembas' :
            #create a healing potion
                item_component = Item(weight = 0.5,use_function=cast_heal)            
                item = Object(x, y, '^', 'Lembas Bread (0.5kg)', libtcod.dark_green,0, item=item_component,always_visible = True)
                                
            elif choice == 'Throwing Axe':
                item_component = Item(weight = 2, use_function=cast_throw) 
                item = Object(x, y, '#', 'Throwing Axe (2kg)', libtcod.light_yellow,0, item=item_component,always_visible = True)       

            elif choice == 'scroll of stun' :    
                item_component = Item(weight = 1, use_function=cast_stun) 
                item = Object(x, y, '>', 'scroll of stun', libtcod.yellow,0, item=item_component, always_visible = True)              
                                
##RUNES##                
            elif choice == 'DwarvenRune':
                rune_component = rune(slot = 'SELF' , type= 'Dwarven')            
                item_component = Item(weight = 1, use_function=None)
                item = Object(x,y,int(205), 'Dwarven rune' , libtcod.amber,0, item=item_component, rune=rune_component, always_visible = True)
                
            elif choice == 'ElvenRune':
                rune_component = rune(slot = 'SELF' , type= 'Elven')            
                item_component = Item(weight = 1, use_function=None)
                item = Object(x,y,int(205), 'Elven rune' , libtcod.lime,0, item=item_component, rune=rune_component, always_visible = True)            
                
            elif choice == 'BeastRune':
                rune_component = rune(slot = 'SELF' , type= 'Beast')            
                item_component = Item(weight = 1, use_function=None)
                item = Object(x,y,int(205), 'Beast rune' , libtcod.dark_green,0, item=item_component, rune=rune_component, always_visible = True)                
                
            elif choice == 'DrakeRune':
                rune_component = rune(slot = 'SELF' , type= 'Drake')            
                item_component = Item(weight = 1, use_function=None)
                item = Object(x,y,int(205), 'Drake rune' , libtcod.flame,0, item=item_component, rune=rune_component, always_visible = True)                
                
            elif choice == 'CorruptRune':
                rune_component = rune(slot = 'SELF' , type= 'Corrupt')            
                item_component = Item(weight = 1, use_function=None)
                item = Object(x,y,int(205), 'Corrupt rune' , libtcod.light_sea, 0,item=item_component, rune=rune_component, always_visible = True)                

            elif choice == 'UndeadRune':
                rune_component = rune(slot = 'SELF' , type= 'Undead')            
                item_component = Item(weight = 1, use_function=None)
                item = Object(x,y,int(205), 'Undead rune' , libtcod.white,0, item=item_component, rune=rune_component, always_visible = True)

            elif choice == 'EvilRune':
                rune_component = rune(slot = 'SELF' , type= 'Evil')            
                item_component = Item(weight = 1, use_function=None)
                item = Object(x,y,int(205), 'Evil rune' , libtcod.dark_crimson,0, item=item_component, rune=rune_component, always_visible = True)

            elif choice == 'ValarRune':
                rune_component = rune(slot = 'SELF' , type= 'Valar')            
                item_component = Item(weight = 1, use_function=None)
                item = Object(x,y,int(205), 'Valar rune' , libtcod.light_violet,0, item=item_component, rune=rune_component, always_visible = True)
                
            elif choice == 'ManRune':
                rune_component = rune(slot = 'SELF' , type= 'Men')            
                item_component = Item(weight = 1, use_function=None)
                item = Object(x,y,int(205), 'rune of Men' , libtcod.light_blue,0, item=item_component, rune=rune_component, always_visible = True)                
                
 ##SWORDS##
            elif choice == 'sword':
                #create a sword               
                               
                equipment_component = Equipment(slot='right hand', power_bonus = 2 + int(weapon_var.get_bonus_var()))       
                item_component = Item(weight = 4 + int(weapon_var.get_weight_var()) ,use_function= None)                   
                item = Object(x, y, '$', weapon_var.get_sword() + '(' + str(item_component.weight) + 'kg)'  
                + 'OF: ' + str(equipment_component.power_bonus), libtcod.sky,0,item=item_component, equipment=equipment_component,always_visible = True)

 ##Hammers##
            elif choice == 'dwarvenhammer':
                #create a sword               

                level_component = Weapon_Level('Orcrist', 0.5 , 0 , 100 , wpn_bonus = 2 , wpn_level= 8 , id_function = id_hammer )                               
                equipment_component = Equipment(slot='right hand', power_bonus = 2 + int(weapon_var.get_bonus_var()) )                
                item_component = Item(weight = 4 + int(weapon_var.get_weight_var()) ,use_function= None)   
                item = Object(x, y, int(185), 'dwarven hammer' + '(' + str(item_component.weight) + 'kg)'  
                + 'OF: ' + str(equipment_component.power_bonus), libtcod.sky,0,item=item_component, equipment=equipment_component , weapon_level = level_component ,always_visible = True)       

                
 ##AXES##
            elif choice == 'woodcuttersaxe':
                #create a sword               
                               
                equipment_component = Equipment(slot='right hand', power_bonus = 2 + int(weapon_var.get_bonus_var()))       
                item_component = Item(weight = 4 + int(weapon_var.get_weight_var()) ,use_function= None)                   
                item = Object(x, y, int(202), "Wood Cutter's Axe" + '(' + str(item_component.weight) + 'kg)'  
                + 'OF: ' + str(equipment_component.power_bonus), libtcod.sky,0, item=item_component, equipment=equipment_component,always_visible = True)    

 ##Maces##
            elif choice == 'orcmace':
                #create a sword               
                               
                equipment_component = Equipment(slot='right hand', power_bonus = 1 + int(weapon_var.get_bonus_var()))       
                item_component = Item(weight = 4 + int(weapon_var.get_weight_var()) ,use_function= None)                   
                item = Object(x, y, int(204), 'orc mace' + '(' + str(item_component.weight) + 'kg)'  
                + 'OF: ' + str(equipment_component.power_bonus), libtcod.sky,0,item=item_component, equipment=equipment_component,always_visible = True)                   
                                
##SHIELDS##                
            elif choice == 'shield':
                #create a shield                          
                equipment_component = Equipment(slot='left hand', defence_bonus=1)              
                item_component = Item(weight = 4,use_function= None)
                item = Object(x, y, ',', 'shield ('  +str(item_component.weight)+ 'kg)', libtcod.azure,0,item=item_component, equipment=equipment_component,always_visible = True)           
                
##HELMETS##                
            elif choice == 'helm':
                equipment_component = Equipment(slot='head', defence_bonus=10)              
                item_component = Item(weight = 2, use_function=None) 
                item = Object(x, y, int(188), 'helm (2kg)', libtcod.light_violet,0, item=item_component,equipment=equipment_component,always_visible = True)      
                
##ARMOUR## 
            elif choice == 'steelplates':
                #create a sword                              
                equipment_component = Equipment(slot='chest', defence_bonus=10)       
                item_component = Item(weight = 6,use_function= None)                   
                item = Object(x, y, int(187), 'steel plating (6kg)' , libtcod.dark_azure,0,item=item_component, equipment=equipment_component,always_visible = True)
                
##GLOVES##                
            elif choice == 'gloves':
                #create a shield                          
                equipment_component = Equipment(slot='bothhands', defence_bonus=1)              
                item_component = Item(weight = 1,use_function= None)
                item = Object(x, y, int(203), 'gloves (1kg)', libtcod.light_crimson,0,item=item_component, equipment=equipment_component,always_visible = True)      
                
##LEGGINGS##
            elif choice == 'steelleggings':
                equipment_component = Equipment(slot='legs', defence_bonus=5)              
                item_component = Item(weight = 2, use_function=None) 
                item = Object(x, y, int(201), 'steel leggings (2kg)', libtcod.light_violet, 0,item=item_component,equipment=equipment_component,always_visible = True)                
 
##BOOTS##
            elif choice == 'boots':
                #create a sword                              
                equipment_component = Equipment(slot='feet', defence_bonus=10)       
                item_component = Item(weight = 1.5,use_function= None)                   
                item = Object(x, y, int(200), 'Boots (1.5kg)' , libtcod.light_crimson,0,item=item_component, equipment=equipment_component,always_visible = True)
                
##NeckLaces##                
            elif choice == 'amethyst':
                #create a shield                          
                equipment_component = Equipment(slot='neck', luck_bonus=1)              
                item_component = Item(weight = 1,use_function= None)
                item = Object(x, y, int(206), 'amethyst (1kg)', libtcod.light_violet,0,item=item_component, equipment=equipment_component,always_visible = True) 
             
##LIGHTSOURCES##
            elif choice == 'Lamp of Ethu':
                equipment_component = Equipment(slot='left hand', light_bonus = 3, step_lives = 20)
                item_component = Item(weight = 2,use_function= None)
                item = Object(x, y, '-', 'Lamp of Ethu', libtcod.sky,0,item=item_component, equipment=equipment_component, always_visible = True)   
             
            objects.append(item)
            item.send_to_back()  #items appear below other objects
           

######
#Weapon Identify#
######           
def id_hammer(item):
    item.name =  str( item.weapon_level.id_name )

    
#############
#GUI GUI GUI#            
#############
def render_bar(x, y, total_width, name, value, maximum, bar_color, back_color):
    #render a bar (HP, experience, etc). first calculate the width of the bar
    bar_width = int(float(value) / maximum * total_width)
 
    #render the background first
    libtcod.console_set_default_background(panel, back_color)
    libtcod.console_rect(panel, x, y, total_width, 1, False, libtcod.BKGND_SCREEN)
 
    #now render the bar on top
    libtcod.console_set_default_background(panel, bar_color)
    if bar_width > 0:
        libtcod.console_rect(panel, x, y, bar_width, 1, False, libtcod.BKGND_SCREEN)
 
    #finally, some centered text with the values
    libtcod.console_set_default_foreground(panel, libtcod.white)
    libtcod.console_print_ex(panel, x + total_width / 2, y, libtcod.BKGND_NONE, libtcod.CENTER,
        name + ': ' + str(value) + '/' + str(maximum))            
        
def render_bar_right(x, y, total_width, name, value, maximum, bar_color, back_color):
    global mouse
    #render a bar (HP, experience, etc). first calculate the width of the bar
    bar_width = int(float(value) / maximum * total_width)
 
    #render the background first
    libtcod.console_set_default_background(panel_right, back_color)
    libtcod.console_rect(panel_right, x, y, total_width, 1, False, libtcod.BKGND_SCREEN)
 
    #now render the bar on top
    libtcod.console_set_default_background(panel_right, bar_color)
    if bar_width > 0:
        libtcod.console_rect(panel_right, x, y, bar_width, 1, False, libtcod.BKGND_SCREEN)
        
 
    #finally, some centered text with the values
    libtcod.console_set_default_foreground(panel_right, libtcod.white)
    libtcod.console_print_ex(panel_right, x + total_width / 2, y, libtcod.BKGND_NONE, libtcod.CENTER,
        name + ': ' + str(value) + '/' + str(maximum))                  
        
def get_names_under_mouse():
    global mouse
 
    #return a string with the names of all objects under the mouse
    (x, y) = (mouse.cx, mouse.cy)
    
    #create a list with the names of all objects at the mouse's coordinates and in FOV
    names = [obj.name for obj in objects
        if obj.x == x and obj.y == y and libtcod.map_is_in_fov(fov_map, obj.x, obj.y)]
        
    names = ', '.join(names)  #join the names, separated by commas
    return  names.capitalize()    
    
########
#Render#
########
def render_all():
    global fov_map, color_dark_wall, color_light_wall
    global color_dark_ground, color_light_ground
    global fov_recompute , turn_step , EP_pool , regen , turn_step_level
    show_step = 0
    
    for equipment in get_all_equipped(player):
        if equipment.step_lives != 0:
            show_step = equipment.step_lives   
        else:
            show_step = 0
 
    if fov_recompute:
        #recompute FOV if needed (the player moved or something)
        fov_recompute = False
        libtcod.map_compute_fov(fov_map, player.x, player.y, TORCH_RADIUS, FOV_LIGHT_WALLS, FOV_ALGO)

        #go through all tiles, and set their background colour according to the FOV
        for y in range(MAP_HEIGHT):
            for x in range(MAP_WIDTH): 
                visible = libtcod.map_is_in_fov(fov_map, x, y)
                wall = map[x][y].block_sight
                if not visible:
                    #if it's not visible right now, the player can only see it if it's explored
                    if map[x][y].explored:                
                    #it's out of the player's FOV
                        if wall:
                            libtcod.console_set_char_background(con, x, y, color_dark_wall, libtcod.BKGND_SET )
                        else:
                            libtcod.console_set_char_background(con, x, y, color_dark_ground, libtcod.BKGND_SET )
                else:
                    #it's visible
                    if wall:
                        libtcod.console_set_char_background(con, x, y, color_light_wall, libtcod.BKGND_SET )
                    else:
                        libtcod.console_put_char_ex(con, x, y, '.', libtcod.grey, libtcod.black)
                    #since it's visible, explore it
                    map[x][y].explored = True
                        
    #draw all objects in the list, except the player. we want it to
    #always appear over all other objects! so it's drawn later.
    for object in objects:
        if object != player:
            object.draw()
    player.draw()
        
    #blit the contents of "con" to the root console and present it
    libtcod.console_blit(con, 0, 0, SCREEN_WIDTH, SCREEN_HEIGHT, 0, 0, 0)
    
    #prepare to render the GUI panel
    libtcod.console_set_default_background(panel, libtcod.black)
    libtcod.console_clear(panel)
    
    #print the game messages, one line at a time
    y = 2
    for (line, color) in game_msgs:
        libtcod.console_set_default_foreground(panel, color)
        libtcod.console_print_ex(panel, MSG_X, y, libtcod.BKGND_NONE, libtcod.LEFT, line)
        y += 1    
    
    #show the player's stats

    
    libtcod.console_set_default_foreground(panel, libtcod.grey)      
    libtcod.console_print_frame(panel, 0, 1, BAR_WIDTH + 2, 7 , clear=True, flag=libtcod.BKGND_NONE, fmt=0)    
    
    libtcod.console_set_default_foreground(panel, libtcod.white)     
    libtcod.console_print_ex(panel, 1, 2, libtcod.BKGND_NONE, libtcod.LEFT, 'EP: ' + str(EP_pool))                    
    libtcod.console_print_ex(panel, 1, 3, libtcod.BKGND_NONE, libtcod.LEFT, 'Depth ' + str(dungeon_level* 20) + 'm')        
    libtcod.console_print_ex(panel, 1, 4, libtcod.BKGND_NONE, libtcod.LEFT, 'Turn Count ' + str(turn_step))    
    libtcod.console_print_ex(panel, 1, 5, libtcod.BKGND_NONE, libtcod.LEFT, 'This Depth ' + str(turn_step_level))     
    
    if show_step == 0:
        libtcod.console_set_default_foreground(panel, libtcod.red)  
    libtcod.console_print_ex(panel, 1, 6, libtcod.BKGND_NONE, libtcod.LEFT, 'Lamp ' + str(show_step))         
    #display names of objects under the mouse
    libtcod.console_set_default_foreground(panel, libtcod.light_gray)
    libtcod.console_print_ex(panel, 1, 0, libtcod.BKGND_NONE, libtcod.LEFT, get_names_under_mouse()) 
 
    #blit the contents of "panel" to the root console
    libtcod.console_blit(panel, 0, 0, SCREEN_WIDTH, PANEL_HEIGHT, 0, 0, PANEL_Y)
    show_keys()  
    
##########
#Messages#    
##########    
def message(new_msg, color = libtcod.white):
    #split the message if necessary, among multiple lines
    new_msg_lines = textwrap.wrap(new_msg, MSG_WIDTH)
 
    for line in new_msg_lines:
        #if the buffer is full, remove the first line to make room for the new one
        if len(game_msgs) == MSG_HEIGHT:
            del game_msgs[0]
 
        #add the new line as a tuple, with the text and the color
        game_msgs.append( (line, color) )    
    
## Move Or Attack? ##    
    
def player_move_or_attack(dx, dy):
    global fov_recompute
 
    #the coordinates the player is moving to/attacking
    x = player.x + dx
    y = player.y + dy
 
    #try to find an attackable object there
    target = None
    for object in objects:
        if object.fighter and object.x == x and object.y == y:
            target = object
            break
 
    #attack if target found, move otherwise
    if target is not None:
        player.fighter.attack(target)
    else:
        player.move(dx, dy)
        fov_recompute = True
        
def player_move(dx, dy):
    global fov_recompute
 
    #the coordinates the player is moving to/attacking
    x = player.x + dx
    y = player.y + dy
    
    target = None
    if target is None:
        player.move(dx, dy)
        fov_recompute = True
        message (name_fetch.got_name() + ' the ' + name_fetch.got_h() + " waits for a moment.")
        
def menu(header, options, width):
    if len(options) > 26: raise ValueError('Cannot have a menu with more than 26 options.')
    
    #calculate total height for the header (after auto-wrap) and one line per option
    header_height = libtcod.console_get_height_rect(con, 0, 0, width, SCREEN_HEIGHT, header)
    if header == '':
        header_height = 0
    height = len(options) + header_height
    
    #create an off-screen console that represents the menu's window
    window = libtcod.console_new(width, height)
 
    #print the header, with auto-wrap
    libtcod.console_set_default_foreground(window, libtcod.white)
    libtcod.console_print_rect_ex(window, 0, 0, width, height, libtcod.BKGND_NONE, libtcod.LEFT, header)    

    #print all the options
    y = header_height
    letter_index = ord('a')
    for option_text in options:
        text = '(' + chr(letter_index) + ') ' + option_text
        libtcod.console_print_ex(window, 0, y, libtcod.BKGND_NONE, libtcod.LEFT, text)
        y += 1
        letter_index += 1
        
    #blit the contents of "window" to the root console
    x = SCREEN_WIDTH/2 - width/2
    y = SCREEN_HEIGHT/2 - height/2
    libtcod.console_blit(window, 0, 0, width, height, 0, x, y, 1.0, 0.7)
    
    #present the root console to the player and wait for a key-press
    libtcod.console_flush()
    key = libtcod.console_wait_for_keypress(True)
 
    if key.vk == libtcod.KEY_ENTER and key.lalt:  #(special case) Alt+Enter: toggle fullscreen
        libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())
 
    #convert the ASCII code to an index; if it corresponds to an option, return it
    index = key.c - ord('a')
    if index >= 0 and index < len(options): return index
    return None
    
def page_menu(header, options, width):
    if len(options) > 26: raise ValueError('Cannot have a menu with more than 26 options.')
    
    #calculate total height for the header (after auto-wrap) and one line per option
    header_height = libtcod.console_get_height_rect(con, 0, 0, width, SCREEN_HEIGHT, header)
    if header == '':
        header_height = 0
    height = len(options) + header_height
    
    #present the root console to the player and wait for a key-press
    libtcod.console_flush()
    key = libtcod.console_wait_for_keypress(True)
 
    if key.vk == libtcod.KEY_ENTER and key.lalt:  #(special case) Alt+Enter: toggle fullscreen
        libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())
 
    #convert the ASCII code to an index; if it corresponds to an option, return it
    index = key.c - ord('a')
    if index >= 0 and index < len(options): return index
    return None    
        
def inventory_menu(header):
    global inventory 
    #show a menu with each item of the inventory as an option
    if len(inventory) == 0:
        options = ['Nothing in Inventory.']
    else:
        options = []
        for item in inventory:
            text = item.name
            #show additional information, in case it's equipped
            if item.equipment and item.equipment.is_equipped:
                text = text + ' (on ' + item.equipment.slot + ')'
            
            
            if item.rune and item.rune.is_enchanted:
                text = text + ' (Rune is on ' + item.rune.slot + ')'
            options.append(text)
        
 
    index = menu(header, options, INVENTORY_WIDTH)
    
    #if an item was chosen, return it
    if index is None or len(inventory) == 0: return None
    return inventory[index].item    
    
def msgbox(text, width=50):
    menu(text, [], width)  #use menu() as a sort of "message box"        
    
##################
#PLAYER KEYS#
##################
def handle_keys(): 
    global fov_recompute, strike_Kill, record_xp , turn_step_level # strike_Kill random death chant #
    global display
    global take_floor , take_steps
    global small_steps , big_steps
    global TORCH_RADIUS
    global option_fullscreen
     
    #key = libtcod.console_check_for_keypress()  #real-time
    #key = libtcod.console_wait_for_keypress(True)  #turn-based
 
    if key.vk == libtcod.KEY_ENTER and key.lalt:
        #Alt+Enter: toggle fullscreen    
        libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())
        if option_fullscreen == 0:
            option_fullscreen = 1
        else:
            option_fullscreen = 0             

    elif key.vk == libtcod.KEY_ESCAPE:
        display = 0
        return 'exit'  #exit game
        
    if game_state == 'dead':
        dead_keys()
        
    elif game_state == 'playing':
        #movement keys
    #movement keys
        if key.vk == libtcod.KEY_KP8:                       #up
            player_move_or_attack(0,-1)
            fov_recompute = True
 
        elif key.vk == libtcod.KEY_KP2:                     #down
            player_move_or_attack(0,1)
            fov_recompute = True
    
        elif key.vk == libtcod.KEY_KP4:                     #left
            player_move_or_attack(-1,0)
            fov_recompute = True
 
        elif key.vk == libtcod.KEY_KP6:                     #right
            player_move_or_attack(1,0)
            fov_recompute = True
        
        elif  key.vk == libtcod.KEY_KP7:                    #4 other directional buttons below
            player_move_or_attack(-1,-1)
            fov_recompute = True
        
        elif key.vk == libtcod.KEY_KP9:
            player_move_or_attack(1,-1)
            fov_recompute = True
        
        elif  key.vk == libtcod.KEY_KP1:
            player_move_or_attack(-1,1)
            fov_recompute = True
        
        elif key.vk == libtcod.KEY_KP3:
            player_move_or_attack(1,1)
            fov_recompute = True
            
        elif  key.vk == libtcod.KEY_KP5:                   #Do nothing
            player_move(0,0)
            fov_recompute = True          
            
        else:
            #test for other keys
            key_char = chr(key.c)
 
            if key_char == 'g':
                #pick up an item
                for object in objects:  #look for an item in the player's tile
                    if object.x == player.x and object.y == player.y and object.item:
                        object.item.pick_up()
                        break
                        
            if key_char == 'i':
                #show the inventory; if an item is selected, use it
                chosen_item = inventory_menu('Press the key next to an item to use it, or any other to cancel.\n')
                if chosen_item is not None:
                    chosen_item.use()                                   
                    
            if key_char == 'h':
                History_Menu()
                    
            if key_char == 'd':
                #show the inventory; if an item is selected, drop it
                chosen_item = inventory_menu('Press the key next to an item to drop it, or any other to cancel.\n')
                if chosen_item is not None:
                    chosen_item.drop()                 
                    
            if key_char == 'l':
                update_xp()
            
            if key_char == 'k':                
                kills_over_single_play()

            if key_char == 'c':
                #show character information
                Character_Menu()

            if key_char == '<':
                #go down stairs, if the player is on them
                if stairs.x == player.x and stairs.y == player.y:
                    if turn_step_level < small_steps:
                        small_steps = turn_step_level
                    if turn_step_level > big_steps:
                        big_steps = turn_step_level    
                
                    take_steps +=turn_step_level 
                    take_floor += 1
                    
                    turn_step_level = 0
                    next_level()                    
                    
            return 'didnt-take-turn'      
            
        if player.fighter.hp < player.fighter.max_hp/5:
            message (name_fetch.got_name() + ' is in pain...', libtcod.red)   
                    

def check_level_up():
    global EP_pool

    level_up_xp = LEVEL_UP_BASE + player.level * LEVEL_UP_FACTOR
    if player.fighter.xp >= level_up_xp:
        #it is! level up
        EP_pool += 5        
        player.level += 1
        player.fighter.xp -= level_up_xp
        msgbox('Level up too: ' + str(player.level), CHARACTER_SCREEN_WIDTH)
        message(name_fetch.got_name()  + ' is getting stronger! He is now Level: ' + str(player.level), libtcod.cyan)    
        
def check_light():
    
    try:
        for equipment in get_all_equipped(player):                    
            if equipment.light_bonus != 0:
                equipment.step_lives -= 1
            if equipment.step_lives == 0:
                equipment.light_bonus = 0                  
                print equipment.step_lives, ',' , equipment.light_bonus
                
    except:
        return        
        
def update_xp(): 

    global THROW_DAMAGE, TORCH_RADIUS, weight_cap, EP_pool , regen_rate
    global tract_power , tract_defence 
    #see if the player's experience is enough to level-up
    global EP_cost_Throwpower , EP_cost_defence, EP_cost_endurance , EP_cost_hp , EP_cost_luck , EP_cost_power , EP_cost_weightcap
    
    

    choice = menu('Experience Points: '+str(EP_pool)+ '\n\nWhere do you want to focus your skills?\n',
        ['Max Hit Points. (Cost:' + str(EP_cost_hp) + ')',
        'Offence (Cost: ' + str(EP_cost_power) + ')',
        'Defence (Cost: ' + str(EP_cost_defence) + ')',        
        'Adept Throwing (Cost: ' + str(EP_cost_Throwpower) + ')',
        'Endurance (Cost: ' + str(EP_cost_endurance) + ')',
        'Carry Weight (Cost: ' + str(EP_cost_weightcap) + ')',
        'Luck (Cost: ' + str(EP_cost_luck) + ')'
        ], LEVEL_SCREEN_WIDTH)
 
    if choice == 0:
        if EP_pool >= EP_cost_hp:
            player.fighter.base_max_hp += 5
            player.fighter.hp += 5
            EP_pool -= EP_cost_hp
            EP_cost_hp += 5
            message( name_fetch.got_name() + ' increases Max Hit Points to '  + str(player.fighter.max_hp) , libtcod.cyan)
        else: 
            message( name_fetch.got_name() + ' can not afford to raise Hit Points' , libtcod.red)
                
    if choice == 1:
        if EP_pool >= EP_cost_power:
            player.fighter.base_power += 1
            tract_power += 1
            EP_pool -= EP_cost_power
            EP_cost_power += 5
            message( name_fetch.got_name() + ' increases Offence to '  + str(player.fighter.power) , libtcod.cyan)
        else: 
            message( name_fetch.got_name() + ' can not afford to raise Offence' , libtcod.red)            

    if choice == 2:
        if EP_pool >= EP_cost_defence: 
            player.fighter.base_defence += 1
            tract_defence += 1
            EP_pool -= EP_cost_defence
            EP_cost_defence += 5
            message( name_fetch.got_name() + ' increases Defence to '  + str(player.fighter.defence) , libtcod.cyan)
        else: 
            message( name_fetch.got_name() + ' can not afford to raise Defence' , libtcod.red)
            
    if choice == 3:
        if EP_pool >= EP_cost_Throwpower:    
            THROW_DAMAGE +=1  
            EP_pool -= EP_cost_Throwpower
            EP_cost_Throwpower += 5   
            message( name_fetch.got_name() + ' increases Throwing Proficiency to '  + str(THROW_DAMAGE -1) , libtcod.cyan)
        else: 
            message( name_fetch.got_name() + ' can not afford to raise Thowing Proficiency' , libtcod.red)
                        
            
    if choice == 4:
        if regen_rate <= 10:
            message( name_fetch.got_name() + 'Endurance is maxed out at Level '  + str(REGEN_LEVEL) , libtcod.red)
        elif EP_pool >= EP_cost_endurance:    
            regen_rate -=1      
            EP_pool -= EP_cost_endurance
            EP_cost_endurance += 5
            message( name_fetch.got_name() + ' increases Endurance to '  + str(REGEN_LEVEL) , libtcod.cyan)
        else: 
            message( name_fetch.got_name() + ' can not afford to raise Endurance' , libtcod.red)
            
    if choice == 5:
        if EP_pool >= EP_cost_weightcap:    
            weight_cap += 5      
            EP_pool -= EP_cost_weightcap
            EP_cost_weightcap += 5
            message( name_fetch.got_name() + ' can now carry '  + str(weight_cap) , libtcod.cyan)
        else: 
            message( name_fetch.got_name() + ' can not afford to raise Carrying Capacity' , libtcod.red)
                                                                   
    if choice ==6:
        if EP_pool >= EP_cost_luck:    
            player.fighter.base_luck += 1 
            EP_pool -= EP_cost_luck
            EP_cost_luck += 5
            message( name_fetch.got_name() + ' increases Luck to '  + str(player.fighter.luck) , libtcod.cyan)          
        else: 
            message( name_fetch.got_name() + ' can not afford to raise Luck' , libtcod.red)            
         
def dead_keys():
      
    if game_state == 'dead':    
        key_char = chr(key.c)    
        
        if key_char == 'k':     #Must be the same as the one under player keys!#
            kills_over_single_play()
            
        if key_char == 'h':
            History_Menu()            
            
        if key_char == 'c':
            Character_Menu()
        
def player_death(player):
    #the game ended!
    global game_state , turn_step_level , take_steps_n
    
    dead_keys()
    message(name_fetch.got_name() + ' died!', libtcod.red)
    game_state = 'dead'
 
    #for added effect, transform the player into a corpse!
    player.char = '%'
    player.color = libtcod.red   
    
    take_steps_n += turn_step_level    
    save_stat()
            
def corrupt_death(monster):
    global warg_kill , warg_pup_kill , mewlip_kill , snaga_kill , snuffler_kill , wargleader_kill
    #transform it into a nasty corpse! it doesn't block, can't be
    #attacked and doesn't move
        
    if monster.name == str('warg pup'):
        warg_pup_kill += 1
        
    if monster.name == str('warg'):
        warg_kill += 1
                
    if monster.name == str('snaga'):
        snaga_kill += 1
        
    if monster.name == str('snuffler'):
        snuffler_kill += 1
        
    if monster.name == str('warg leader'):
        wargleader_kill += 1
        
    if monster.name == str('mewlip'):
        mewlip_kill += 1
        x = monster.x
        y = monster.y
        rune_component = rune(slot = 'SELF' , type= 'Men')            
        item_component = Item(weight = 1, use_function=None)
        item = Object(x,y,int(205), 'rune of Men' , libtcod.light_blue, 0, item=item_component, rune=rune_component, always_visible = True)     
        objects.append(item)
        item.send_to_back()        
        
        
    message ('The ' + monster.name + Kill_calls.corrupt_kcall(), libtcod.orange)
    message(str(monster.fighter.xp) + ' experience points.', libtcod.light_green)
    monster.char = '%'
    monster.color = libtcod.dark_red
    monster.blocks = False
    monster.fighter = None
    monster.ai = None
    monster.name = monster.name + ' remains.'
    monster.send_to_back()    
    
def beast_death(monster):
    global rabbit_kill , crebain_kill , wolf_kill , greywolf_kill
    
    if monster.name == str('rabbit'):  
        rabbit_kill += 1              
    if monster.name == str('crebain'):  
        crebain_kill += 1
    if monster.name == str('wolf'):  
        wolf_kill += 1
    if monster.name == str('greywolf'):  
        greywolf_kill += 1      
        
    message ('The ' +  monster.name + Kill_calls.beast_kcall() , libtcod.orange)                
    message(str(monster.fighter.xp) + ' experience points.', libtcod.light_green)
    monster.char = '%'
    monster.color = libtcod.dark_red
    monster.blocks = False
    monster.fighter = None
    monster.ai = None
    
    if Kill_calls.beast_kcall() == str( " is smashed into an unrecognisable mush!"):
        monster.name = 'mush'
    else:
        monster.name = monster.name + ' remains.'
    monster.send_to_back()        
    
def undead_death(monster):
    global skeleton_kill , skeletonnewb_kill
    
    if monster.name == str('skeleton'):  
        skeletonnewb_kill += 1
    if monster.name == str('skeleton soldier'):  
        skeleton_kill += 1     

    message ('The ' +  monster.name + Kill_calls.undead_kcall() , libtcod.orange)                
    message(str(monster.fighter.xp) + ' experience points.', libtcod.light_green)
    monster.char = '%'
    monster.color = libtcod.white
    monster.blocks = False
    monster.fighter = None
    monster.ai = None

    monster.name =  'Pile of limbs and bones.'
    monster.send_to_back()            
    
def evil_death(monster):
    global spectre_kill , spectrenewb_kill
    if monster.name == str('spectre'):  
        spectrenewb_kill += 1
    if monster.name == str('royal spectre'):  
        spectre_kill += 1  

    message ('The ' +  monster.name + Kill_calls.evil_kcall() , libtcod.orange)                
    message(str(monster.fighter.xp) + ' experience points.', libtcod.light_green)
    monster.char = '.'
    monster.color = libtcod.white
    monster.blocks = False
    monster.fighter = None
    monster.ai = None

    monster.name =  'Evil has been vanquished.'
    monster.send_to_back()         
    
def box_death(monster):
    if monster.name == str('chest'):
    
        x = monster.x
        y = monster.y
        rune_component = rune(slot = 'SELF' , type= 'Men')            
        item_component = Item(weight = 1, use_function=None)
        item = Object(x,y,int(205), 'rune of Men' , libtcod.light_blue,0, item=item_component, rune=rune_component, always_visible = True)     
        objects.append(item)
        
        message ('The ' +  monster.name + ' has been opened.' , libtcod.orange)                
        message(str(monster.fighter.xp) + ' experience points.', libtcod.light_green)
        monster.char = int(225)
        monster.color = libtcod.light_azure
        monster.blocks = False
        monster.fighter = None
        monster.ai = None
        monster.send_to_back()       
        monster.name =  'An Opened Container.'     
        
    if monster.name == str('tomb'):
        tomb_spawn = libtcod.random_get_int(0,0,100)
        if tomb_spawn < 70:        
            skeleton_var = libtcod.random_get_int(0,0,4)
            x = monster.x
            y = monster.y        
            fighter_component = Fighter("Undeadmob",hp= 2+skeleton_var , defence=0, power=0, xp = 50, luck = 2, death_function= undead_death)
            ai_component = WanderAI()
            monster = Object(x, y, 's', "skeleton", libtcod.white,0, blocks=True, fighter=fighter_component, ai=ai_component)       
            objects.append(monster)     
            message ('A ' +  monster.name + ' crawls out the tomb!' , libtcod.orange)  

        elif tomb_spawn < 90:
            message ('There is nothing in here.' , libtcod.orange)                
            message(str(monster.fighter.xp) + ' experience points.', libtcod.light_green)
            monster.char = int(225)
            monster.color = libtcod.light_azure
            monster.blocks = False
            monster.fighter = None
            monster.ai = None
            monster.send_to_back()       
            monster.name =  'An Opened Container.'      

        elif tomb_spawn <= 100:
            x = monster.x
            y = monster.y
            rune_component = rune(slot = 'SELF' , type= 'Undead')            
            item_component = Item(weight = 1, use_function=None)
            item = Object(x,y,int(205), 'Undead rune' , libtcod.white,0, item=item_component, rune=rune_component, always_visible = True)  
            objects.append(item)
            
            message ('The ' +  monster.name + ' has been opened.' , libtcod.orange)                
            message(str(monster.fighter.xp) + ' experience points.', libtcod.light_green)
            monster.char = int(225)
            monster.color = libtcod.light_azure
            monster.blocks = False
            monster.fighter = None
            monster.ai = None
            monster.send_to_back()       
            monster.name =  'An Opened Container.'             

def target_monster(max_range=None):
    #returns a clicked monster inside FOV up to a range, or None if right-clicked
    while True:
        (x, y) = target_tile(max_range)
        if x is None:  #player cancelled
            return None
 
        #return the first clicked monster, otherwise continue looping
        for obj in objects:
            if obj.x == x and obj.y == y and obj.fighter and obj != player:
                return obj            
    
def target_tile(max_range=None):
    #return the position of a tile left-clicked in player's FOV (optionally in a range), or (None,None) if right-clicked.
    global key, mouse
    while True:
        #render the screen. this erases the inventory and shows the names of objects under the mouse.
        libtcod.console_flush()
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS|libtcod.EVENT_MOUSE,key,mouse)
        render_all()
 
        (x, y) = (mouse.cx, mouse.cy)
        if mouse.rbutton_pressed or key.vk == libtcod.KEY_ESCAPE:
            return (None, None)  #cancel if the player right-clicked or pressed Escape        
 
        if (mouse.lbutton_pressed and libtcod.map_is_in_fov(fov_map, x, y) and
            (max_range is None or player.distance(x, y) <= max_range)):
            return (x, y)

def cast_heal():
    #heal the player
    if player.fighter.hp == player.fighter.max_hp:
        message(name_fetch.got_name() +' eats the lembas bread.', libtcod.light_violet)
        player.fighter.heal(0)
        
    else:
        message( 'The bread makes ' +name_fetch.got_name() + ' feel better.', libtcod.light_violet)
        player.fighter.heal(HEAL_AMOUNT)    
        
def cast_stun():
    STUN_RANGE= TORCH_RADIUS
    #ask the player for a target to confuse
    message('Left-click an enemy to stun it, or right-click to cancel.', libtcod.light_cyan)
    monster = target_monster(STUN_RANGE)
    if monster is None: return 'cancelled'
        
    #replace the monster's AI with a "confused" one; after some turns it will restore the old AI
    old_ai = monster.ai
    monster.ai = StunnedMonster(old_ai)
    monster.ai.owner = monster  #tell the new component who owns it
    message('The eyes of the ' + monster.name + ' look vacant, as he starts to stumble around!', libtcod.light_green)        
    
def cast_throw():
    global fov_recompute
    #ask the player for a target tile to throw a fireball at
    message('Select Target right-click to cancel.', libtcod.light_cyan)
    (x, y) = target_tile()
    if x is None: return 'cancelled'    
    
    break_strike = libtcod.random_get_int(0,0,100)
    
    if break_strike < 5:
        message(name_fetch.got_name() + "'s might shatters the Axe!" , libtcod.orange)
        
        for obj in objects:  #damage every fighter in range, including the player
            if obj.distance(x, y) <= THROW_RADIUS and obj.fighter:
                message('The ' + obj.name + ' gets hit for ' + str(THROW_DAMAGE*2) + ' hit points.', libtcod.orange)
                obj.fighter.take_damage(THROW_DAMAGE * 2)    
        
    else:
        message(name_fetch.got_name() +' throws the Axe!', libtcod.orange)
        item_component = Item(weight = 2, use_function=cast_throw) 
        item = Object(x, y, '#', 'Throwing Axe (2kg)', libtcod.light_yellow,0, item=item_component,always_visible = True)   
        objects.append(item)
        item.send_to_back()
        
        for obj in objects:  #damage every fighter in range, including the player
            if obj.distance(x, y) <= THROW_RADIUS and obj.fighter:
                message('The ' + obj.name + ' gets hit for ' + str(THROW_DAMAGE) + ' hit points.', libtcod.orange)
                obj.fighter.take_damage(THROW_DAMAGE)    
               

    
#############################################
# Initialization & Main Loop
#############################################
def save_game():

    #open a new empty shelve (possibly overwriting an old one) to write the game data
    file = shelve.open('savegame' , 'n')
    file['tract_power'] = tract_power  
    file['tract_defence'] = tract_defence        
    file['REGEN_LEVEL'] = REGEN_LEVEL
    file['regen_rate'] = regen_rate
    file['map'] = map
    file['objects'] = objects
    file['player_index'] = objects.index(player)  #index of player in objects list
    file['inventory'] = inventory
    file['game_msgs'] = game_msgs
    file['game_state'] = game_state
    file['weight_inventory'] = weight_inventory
    file['stairs_index'] = objects.index(stairs)
    file['dungeon_level'] = dungeon_level 
    
    file.close()
    
def load_game():
    #open the previously saved shelve and load the game data
    global map, objects, player, inventory, game_msgs, game_state
    global stairs, dungeon_level, tract_power, tract_defence
    global REGEN_LEVEL, regen_rate
    
    file = shelve.open('savegame' , 'r')

    tract_power = file['tract_power']         
    tract_defence = file['tract_defence']     
    REGEN_LEVEL = file['REGEN_LEVEL']
    regen_rate = file['regen_rate']
    map = file['map']
    objects = file['objects']
    player = objects[file['player_index']]  #get index of player in objects list and access it
    inventory = file['inventory']
    stairs = objects[file['stairs_index']]
    dungeon_level = file['dungeon_level']
    game_msgs = file['game_msgs']
    game_state = file['game_state']

    file.close()
 
    initialize_fov()
    
def save_stat():
    file = shelve.open('savestat' , 'n')
    file['take_steps'] = take_steps 
    file['take_floor'] = take_floor 
    file['take_games'] = take_games     
    file['small_steps'] = small_steps  
    file['big_steps'] = big_steps    
    file['take_steps_n'] = take_steps_n      
    
    
    file.close()    

def load_stat():    
    global take_steps , take_floor , take_games , small_steps , big_steps , take_steps_n
    
    file = shelve.open('savestat' , 'r')    
    take_steps = file['take_steps']    
    take_floor = file['take_floor']    
    take_games = file['take_games']      
    small_steps = file['small_steps']  
    big_steps = file['big_steps']      
    take_steps_n = file['take_steps_n']      
    
    file.close()    
        
    
def new_game():
    global player, inventory, game_msgs, game_state, dungeon_level , turn_step , turn_step_level
        
    #create object representing the player
    fighter_component = Fighter('Mortal',hp=5, defence=1, power=1, xp=0,luck = 0, death_function=player_death) 
    player = Object(0, 0, '@', 'The ' + name_fetch.got_h() + name_fetch.got_m(), libtcod.white,0, blocks=True, fighter=fighter_component)

    
    player.level = 1        
    dungeon_level = 1
    make_map()
    initialize_fov()
    
    game_state = 'playing'
    inventory = []

    #create the list of game messages and their colors, starts empty
    game_msgs = []
    
    #a warm welcoming message!

    message('Welcome back ' + name_fetch.got_h() + name_fetch.got_m() + ' ' + name_fetch.got_name() + '!' + Dungeon_story.set_story() , libtcod.light_grey)

    #initial equipment: a dagger
    equipment_component = Equipment(slot='left hand', light_bonus = 1, step_lives = 600)
    item_component = Item(weight = 1,use_function= None)
    obj = Object(0, 0, '-', 'lamp', libtcod.sky,0,item=item_component, equipment=equipment_component)
    inventory.append(obj)
    equipment_component.equip()
    obj.always_visible = True    

    
def next_level():
    global dungeon_level
    global take_steps , take_floor
    #advance to the next level
    take_floor += 1
    
    turn_step_level = 0
    dungeon_level += 1
    message(name_fetch.got_name() + Dungeon_story.story_by_level() , libtcod.red)
    make_map()  #create a fresh new level!
    initialize_fov()
    
def initialize_fov():
    global fov_recompute, fov_map
    fov_recompute = True
 
    #create the FOV map, according to the generated map
    fov_map = libtcod.map_new(MAP_WIDTH, MAP_HEIGHT)
    for y in range(MAP_HEIGHT):
        for x in range(MAP_WIDTH):
            libtcod.map_set_properties(fov_map, x, y, not map[x][y].block_sight, not map[x][y].blocked)
 
    libtcod.console_clear(con)  #unexplored areas start black (which is the default background color)
    
def turn_count():
    global turn_step , regen , regen_rate , player , turn_step_level
    
    regen += 1     
    turn_step += 1
    turn_step_level += 1
    
    if regen_rate <= 10:            
        if regen == set_regen and player.fighter.hp < player.fighter.max_hp:
            player.fighter.hp += 1
            regen = 0
        elif player.fighter.hp == player.fighter.max_hp:
            regen = 0
            
    elif regen == regen_rate and player.fighter.hp < player.fighter.max_hp:
        player.fighter.hp += 1
        regen = 0
            
    elif player.fighter.hp == player.fighter.max_hp:
        regen = 0           
              
def regen_level():
    global regen_rate , REGEN_LEVEL , BAR_WIDTH
    
    REGEN_LEVEL = math.fabs(regen_rate - 31)
         
def show_keys():
    global player , target , power , self 
    global tract_power , tract_defence
    global display , display_max_hp , display_damage
    
    libtcod.console_set_default_background(panel_right, libtcod.black)
    libtcod.console_clear(panel_right)
    libtcod.console_set_default_foreground(panel_right, libtcod.grey)      
    libtcod.console_print_frame(panel_right, 0, 0, 20, 55, clear=True, flag=libtcod.BKGND_NONE, fmt=0)
    
    libtcod.console_set_default_foreground(panel_right, libtcod.light_blue)    
    libtcod.console_print_ex(panel_right, 1, 1, libtcod.BKGND_NONE, libtcod.LEFT,'Keys:' )  
    
    libtcod.console_set_default_foreground(panel_right, libtcod.white)        
    libtcod.console_print_ex(panel_right, 2, 2, libtcod.BKGND_NONE, libtcod.LEFT, '[I]nventory' )  
    libtcod.console_print_ex(panel_right, 2, 3, libtcod.BKGND_NONE, libtcod.LEFT, '[G]et / Pick Up' )  
    libtcod.console_print_ex(panel_right, 2, 4, libtcod.BKGND_NONE, libtcod.LEFT, '[D]rop Item' )  
    libtcod.console_print_ex(panel_right, 2, 5, libtcod.BKGND_NONE, libtcod.LEFT, '[K]ill List' )  
    libtcod.console_print_ex(panel_right, 2, 6, libtcod.BKGND_NONE, libtcod.LEFT, '[C]haracter Info' )
    libtcod.console_print_ex(panel_right, 2 , 7, libtcod.BKGND_NONE, libtcod.LEFT,  '[L]evel Up' )       
    libtcod.console_print_ex(panel_right, 2, 8, libtcod.BKGND_NONE, libtcod.LEFT, '[H]istory' )
    libtcod.console_print_ex(panel_right, 2 , 9, libtcod.BKGND_NONE, libtcod.LEFT,  '[<]Down Stairs' )   
    
    libtcod.console_set_default_foreground(panel_right, libtcod.light_blue)     
    libtcod.console_print_ex(panel_right, 1, 11, libtcod.BKGND_NONE, libtcod.LEFT, 'Blessing = ' + str(player.fighter.type) )  
    
    libtcod.console_set_default_foreground(panel_right, libtcod.grey)      
    libtcod.console_print_frame(panel_right, 0, 22-9, BAR_WIDTH  + 2, 5, clear=True, flag=libtcod.BKGND_NONE, fmt=0)    
    
    libtcod.console_print_ex(panel_right, 1, 22-9, libtcod.BKGND_NONE, libtcod.LEFT, str(name_fetch.got_name()) )
    level_up_xp = LEVEL_UP_BASE + player.level * LEVEL_UP_FACTOR    
    
    states = libtcod.Color(100, 220, 50)
    if player.fighter.hp < player.fighter.max_hp:
        states = libtcod.Color(55, 155, 180)
        if player.fighter.hp <= player.fighter.max_hp/5:
            states = libtcod.Color(210, 0, 20)
    render_bar_right(1, 23-9, BAR_WIDTH, 'HP', player.fighter.hp, player.fighter.max_hp,
        states, libtcod.Color(105,15,15))
        
    render_bar_right(1,24-9,BAR_WIDTH, 'INVT', weight_inventory, weight_cap, libtcod.Color(255,220,55), libtcod.darkest_blue)
    render_bar_right(1,25-9, BAR_WIDTH, 'EXP', player.fighter.xp, level_up_xp, libtcod.light_violet, libtcod.darkest_violet)    
    
    if display > 0:
        libtcod.console_set_default_foreground(panel_right, libtcod.red)      
        libtcod.console_print_ex(panel_right, 1 , 28 , libtcod.BKGND_NONE, libtcod.LEFT, 'Target Health:' ) 
        render_bar_right(1, 29, BAR_WIDTH, 'HP', display, display_max_hp, libtcod.darker_red, libtcod.darkest_red)
        libtcod.console_print_ex(panel_right, 1 , 30 , libtcod.BKGND_NONE, libtcod.LEFT, str(name_fetch.got_name()) +"'s damage: " + str(display_damage) ) 
 
   # libtcod.console_print_ex(panel_right, 1, 19, libtcod.BKGND_NONE, libtcod.LEFT, get_weights_under_mouse())               
    
        #blit the contents of "panel" to the root console
    libtcod.console_blit(panel_right, 0, 0, PANEL_RIGHT_W, SCREEN_HEIGHT, 0, MAP_WIDTH, 0)
    
def play_game():
    global key, mouse , TORCH_RADIUS
    
    player_action = None
  
    mouse = libtcod.Mouse()
    key = libtcod.Key()
    while not libtcod.console_is_window_closed():
        #render the screen
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS|libtcod.EVENT_MOUSE,key,mouse)
        render_all()
  
 
        libtcod.console_flush()
        check_level_up() 
        
        #Check lamp/light source
        bonus = sum(equipment.light_bonus for equipment in get_all_equipped(player))
        TORCH_RADIUS = bonus + 2        

        #erase all objects at their old locations, before they move
        for object in objects:
            object.clear()
 
        #handle keys and exit game if needed
        player_action = handle_keys()
        if player_action == 'exit':
            save_game()
            name_fetch.save_game()
            main_menu()
            break
            
        # let monsters take their turn.  
        if game_state == 'playing' and player_action != 'didnt-take-turn':
            turn_count()
            check_light()

            for object in objects:
                if object.ai:
                    object.ai.take_turn()
        show_keys()     
        
def Character_Menu():    
    level_up_xp = LEVEL_UP_BASE + player.level * LEVEL_UP_FACTOR
    img3 = libtcod.image_load('Blank.png') 
    
    base_lane = 50   
    equipt_lane = 59
    total_lane = 72
    while not libtcod.console_is_window_closed():
        libtcod.image_blit_2x(img3, 0, 0, 0)      
        #show the background image, at twice the regular console resolution
        libtcod.console_set_default_background(0, libtcod.black)
        libtcod.console_set_default_foreground(0, libtcod.grey)      
        
        libtcod.console_print_frame(0, 20, 24, 60, 12, clear=True, flag=libtcod.BKGND_NONE, fmt=0)
        libtcod.console_print_frame(0, 32, 15, 36, 9, clear=True, flag=libtcod.BKGND_NONE, fmt=0)        
        libtcod.console_print_frame(0, 32, 8, 36, 7, clear=True, flag=libtcod.BKGND_NONE, fmt=0)
        libtcod.console_print_frame(0, 65, 24, 15, 12, clear=True, flag=libtcod.BKGND_NONE, fmt=0)
              
        #show the game's title, and some credits!
        libtcod.console_set_default_foreground(0, libtcod.white)
        
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-21, libtcod.BKGND_NONE, libtcod.CENTER,
            name_fetch.got_name() + "'s Information." ) 
            
        libtcod.console_set_default_foreground(0, libtcod.light_grey)    
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-19, libtcod.BKGND_NONE, libtcod.CENTER,
            'Identification:' )                                  
        libtcod.console_set_default_foreground(0, libtcod.dark_sky)    
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-17, libtcod.BKGND_NONE, libtcod.CENTER,
            'Name: ' + name_fetch.got_name())                
        libtcod.console_print_ex(0, SCREEN_WIDTH/2 , SCREEN_HEIGHT/2-16, libtcod.BKGND_NONE, libtcod.CENTER,
            'Title: ' + name_fetch.got_h())                
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-15, libtcod.BKGND_NONE, libtcod.CENTER,
            'Profession: ' + name_fetch.got_m() )  
        
        libtcod.console_set_default_foreground(0, libtcod.light_grey)
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-12, libtcod.BKGND_NONE, libtcod.CENTER,
            'Experience:' )                 #
        libtcod.console_set_default_foreground(0, libtcod.dark_sky)    
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-10, libtcod.BKGND_NONE, libtcod.CENTER,
            'Level: ' + str(player.level)  )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-9, libtcod.BKGND_NONE, libtcod.CENTER,
            'Experience: ' +str(player.fighter.xp))  
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-8, libtcod.BKGND_NONE, libtcod.CENTER,
            'Experience to level up: ' +str(level_up_xp))
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-7, libtcod.BKGND_NONE, libtcod.CENTER,
            'Total Experience: ' +str(record_xp)) 
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-6, libtcod.BKGND_NONE, libtcod.CENTER,
            'Experience Points (EP): ' +str(EP_pool))      
            
        libtcod.console_set_default_foreground(0, libtcod.light_grey)
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-3, libtcod.BKGND_NONE, libtcod.CENTER,
            'Attributes:')    
                    
                    
        libtcod.console_set_default_foreground(0, libtcod.white)                        
        libtcod.console_print_ex(0, base_lane, SCREEN_HEIGHT/2-2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Base')            
        libtcod.console_print_ex(0, equipt_lane, SCREEN_HEIGHT/2-2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Equipt')             
        libtcod.console_print_ex(0, total_lane, SCREEN_HEIGHT/2-2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Total')               
            
        libtcod.console_set_default_foreground(0, libtcod.dark_sky)            
        libtcod.console_print_ex(0,32, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Maximum Hit Points:')               
        libtcod.console_print_ex(0, 32, SCREEN_HEIGHT/2+1, libtcod.BKGND_NONE, libtcod.CENTER,
            'Attack:')               
        libtcod.console_print_ex(0, 32, SCREEN_HEIGHT/2+2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Defence:')               
        libtcod.console_print_ex(0, 32, SCREEN_HEIGHT/2+3, libtcod.BKGND_NONE, libtcod.CENTER,
            'Throwing Proficiency:')    
        libtcod.console_print_ex(0,32, SCREEN_HEIGHT/2+4, libtcod.BKGND_NONE, libtcod.CENTER,
            'Level of Endurance:')    
        libtcod.console_print_ex(0, 32, SCREEN_HEIGHT/2+5, libtcod.BKGND_NONE, libtcod.CENTER,
            'Max Carry Weight:')   

        libtcod.console_print_ex(0,32, SCREEN_HEIGHT/2+6, libtcod.BKGND_NONE, libtcod.CENTER,
            'Luck:')              

        libtcod.console_set_default_foreground(0, libtcod.grey)     
        libtcod.console_print_ex(0,base_lane, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
            str(player.fighter.max_hp))      
        libtcod.console_print_ex(0, base_lane, SCREEN_HEIGHT/2+1, libtcod.BKGND_NONE, libtcod.CENTER,
            str(tract_power))               
        libtcod.console_print_ex(0, base_lane, SCREEN_HEIGHT/2+2, libtcod.BKGND_NONE, libtcod.CENTER,
            str(tract_defence))               
        libtcod.console_print_ex(0, base_lane, SCREEN_HEIGHT/2+3, libtcod.BKGND_NONE, libtcod.CENTER,
           str(THROW_DAMAGE - 1))    
        libtcod.console_print_ex(0,base_lane, SCREEN_HEIGHT/2+4, libtcod.BKGND_NONE, libtcod.CENTER,
             str(REGEN_LEVEL))    
        libtcod.console_print_ex(0, base_lane, SCREEN_HEIGHT/2+5, libtcod.BKGND_NONE, libtcod.CENTER,
            str(weight_cap) + 'kg')   

        libtcod.console_print_ex(0,base_lane, SCREEN_HEIGHT/2+6, libtcod.BKGND_NONE, libtcod.CENTER,
            str(player.fighter.luck) )             
            
        libtcod.console_set_default_foreground(0, libtcod.grey)     
        libtcod.console_print_ex(0,equipt_lane, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
            '-')               
        libtcod.console_print_ex(0, equipt_lane, SCREEN_HEIGHT/2+1, libtcod.BKGND_NONE, libtcod.CENTER,
            str(player.fighter.power - tract_power))               
        libtcod.console_print_ex(0, equipt_lane, SCREEN_HEIGHT/2+2, libtcod.BKGND_NONE, libtcod.CENTER,
            str(player.fighter.defence - tract_defence))               
        libtcod.console_print_ex(0, equipt_lane, SCREEN_HEIGHT/2+3, libtcod.BKGND_NONE, libtcod.CENTER,
          '-')    
        libtcod.console_print_ex(0,equipt_lane, SCREEN_HEIGHT/2+4, libtcod.BKGND_NONE, libtcod.CENTER,
             '-')    
        libtcod.console_print_ex(0, equipt_lane, SCREEN_HEIGHT/2+5, libtcod.BKGND_NONE, libtcod.CENTER,
            '-')   
        libtcod.console_print_ex(0,equipt_lane, SCREEN_HEIGHT/2+6, libtcod.BKGND_NONE, libtcod.CENTER,
            '-' )               
            
        libtcod.console_set_default_foreground(0, libtcod.grey)     
        libtcod.console_print_ex(0,total_lane, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
            str(player.fighter.max_hp))               
        libtcod.console_print_ex(0, total_lane, SCREEN_HEIGHT/2+1, libtcod.BKGND_NONE, libtcod.CENTER,
            str(player.fighter.power))               
        libtcod.console_print_ex(0, total_lane, SCREEN_HEIGHT/2+2, libtcod.BKGND_NONE, libtcod.CENTER,
            str(player.fighter.defence))               
        libtcod.console_print_ex(0, total_lane, SCREEN_HEIGHT/2+3, libtcod.BKGND_NONE, libtcod.CENTER,
           str(THROW_DAMAGE - 1))    
        libtcod.console_print_ex(0,total_lane, SCREEN_HEIGHT/2+4, libtcod.BKGND_NONE, libtcod.CENTER,
             str(REGEN_LEVEL))    
        libtcod.console_print_ex(0, total_lane, SCREEN_HEIGHT/2+5, libtcod.BKGND_NONE, libtcod.CENTER,
            str(weight_cap) + 'kg')   
        libtcod.console_print_ex(0,total_lane, SCREEN_HEIGHT/2+6, libtcod.BKGND_NONE, libtcod.CENTER,
            str(player.fighter.luck) )                       
            
        libtcod.console_set_default_foreground(0, libtcod.white)  
        
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT - 3, libtcod.BKGND_NONE, libtcod.CENTER,
            'C) to return to Game')                   
            
        choice = page_menu('', [''], 24)
                        
        if choice == 0:
            break
        else:
            break          
        
def History_Menu():
    img3 = libtcod.image_load('Blank.png') 
    while not libtcod.console_is_window_closed():
        libtcod.image_blit_2x(img3, 0, 0, 0)      
        #show the background image, at twice the regular console resolution
        libtcod.console_set_default_background(0, libtcod.black)
        libtcod.console_set_default_foreground(0, libtcod.grey)      
        
        libtcod.console_print_frame(0, 32, 16, 36, 32, clear=True, flag=libtcod.BKGND_NONE, fmt=0)
        libtcod.console_print_frame(0, 32, 8, 36, 8, clear=True, flag=libtcod.BKGND_NONE, fmt=0)
              
        #show the game's title, and some credits!
        libtcod.console_set_default_foreground(0, libtcod.white)
        
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-21, libtcod.BKGND_NONE, libtcod.CENTER,
            name_fetch.got_name() + "'s Description." ) 
                    
        libtcod.console_set_default_foreground(0, libtcod.light_grey)            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-19, libtcod.BKGND_NONE, libtcod.CENTER,
            'History:')     
            
        libtcod.console_set_default_foreground(0, libtcod.sky)    
        libtcod.console_print_ex(0, SCREEN_WIDTH/2 , SCREEN_HEIGHT/2-18, libtcod.BKGND_NONE, libtcod.CENTER,
            '\n\nGrew up in the Blue Mountains.\nSon of Uther and Daas.\nTrained as a ' + name_fetch.got_m())            
        
        libtcod.console_set_default_foreground(0, libtcod.light_grey)
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-11, libtcod.BKGND_NONE, libtcod.CENTER,
            'About ' + name_fetch.got_name() + ':')  
            
        libtcod.console_set_default_foreground(0, libtcod.sky)    
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-9, libtcod.BKGND_NONE, libtcod.CENTER,
            'Age: ' + str(name_fetch.find_age()))  
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-8, libtcod.BKGND_NONE, libtcod.CENTER,
            'Height: 1.' +str(name_fetch.find_height()))  
            
            
        libtcod.console_set_default_foreground(0, libtcod.white)  
        
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT - 3, libtcod.BKGND_NONE, libtcod.CENTER,
            'H) to return to Game')                   
            
        choice = page_menu('', [''], 24)
                        
        if choice == 0:
            break
        else:
            break        
             
def kills_over_single_play():

    img3 = libtcod.image_load('Blank.png') 
    while not libtcod.console_is_window_closed():
        libtcod.image_blit_2x(img3, 0, 0, 0)      
        #show the background image, at twice the regular console resolution
        libtcod.console_set_default_background(0, libtcod.black)
        libtcod.console_set_default_foreground(0, libtcod.grey)      
        
        libtcod.console_print_frame(0, 32, 13, 36, 35, clear=True, flag=libtcod.BKGND_NONE, fmt=0)
        libtcod.console_print_frame(0, 32, 7, 36, 6, clear=True, flag=libtcod.BKGND_NONE, fmt=0)
              
        #show the game's title, and some credits!
        libtcod.console_set_default_foreground(0, libtcod.orange)
        
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-21, libtcod.BKGND_NONE, libtcod.CENTER,
            'Kills By ' + name_fetch.got_name() + '!' ) 
            
        total_kill = rabbit_kill + warg_pup_kill + skeletonnewb_kill + crebain_kill + skeleton_kill + wolf_kill + mewlip_kill + spectre_kill + greywolf_kill + snaga_kill + warg_kill + spectrenewb_kill + snuffler_kill +wargleader_kill
        total_kill_beasts = greywolf_kill + wolf_kill + crebain_kill + rabbit_kill
        total_kill_corrupt = warg_kill + warg_pup_kill + wargleader_kill + snuffler_kill + mewlip_kill + snaga_kill
        total_kill_undead = skeleton_kill + skeletonnewb_kill
        total_kill_evil = spectre_kill + spectrenewb_kill
        
        
        
        
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-19, libtcod.BKGND_NONE, libtcod.CENTER,
            'Total: ' + str(total_kill))     
            
        if total_kill > 0.5:
            if rabbit_kill < 3:
                pride_str = 'Getting Started'
            elif total_kill - rabbit_kill < rabbit_kill:
                pride_str = 'Rabbit Hater'
            elif total_kill - warg_pup_kill < warg_pup_kill:
                pride_str = 'Baby Killer'
            elif total_kill - total_kill_evil < total_kill_evil:
                pride_str = 'Strong Faith'
            elif total_kill - total_kill_beasts < total_kill_beasts:
                pride_str = 'Beast Hunter'
            elif total_kill - total_kill_corrupt < total_kill_corrupt:
                pride_str = 'Public Protector'
            elif total_kill - total_kill_undead < total_kill_undead:
                pride_str = 'Bone Collector'
            elif total_kill - crebain_kill < crebain_kill:
                pride_str = 'No Flier'
            elif total_kill < 30:
                pride_str = 'Warmed Up'
            elif total_kill <61:
                pride_str = 'Doing Some Good'
            elif total_kill <151:
                pride_str = 'Goner' 
                
            else:
                pride_str = 'Check'
        else:
            pride_str = 'Rogue Newb'
            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2 , SCREEN_HEIGHT/2-18, libtcod.BKGND_NONE, libtcod.CENTER,
            'Pride Status: ' + pride_str)           
            
            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-14, libtcod.BKGND_NONE, libtcod.CENTER,
            'Kills By Name!')         
                        
        libtcod.console_set_default_foreground(0, libtcod.dark_orange)            
        if rabbit_kill > 0:
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-12, libtcod.BKGND_NONE, libtcod.CENTER,
                'Rabbits: ' + str(rabbit_kill))       
        if warg_pup_kill > 0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-11, libtcod.BKGND_NONE, libtcod.CENTER,
                'Warg pups: ' + str(warg_pup_kill)) 
        if skeletonnewb_kill > 0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-10, libtcod.BKGND_NONE, libtcod.CENTER,
                'Skeletons: ' + str(skeletonnewb_kill))  
        if crebain_kill > 0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-9, libtcod.BKGND_NONE, libtcod.CENTER,
                'Crebains: ' + str(crebain_kill))  
        if skeleton_kill > 0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-8, libtcod.BKGND_NONE, libtcod.CENTER,
                'Skeleton Soldiers: ' + str(skeleton_kill))  
        if wolf_kill >0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-7, libtcod.BKGND_NONE, libtcod.CENTER,
                'Wolves: ' + str(wolf_kill))  
        if mewlip_kill > 0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-6, libtcod.BKGND_NONE, libtcod.CENTER,
                'Mewlips: ' + str(mewlip_kill))  
        if spectrenewb_kill > 0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-5, libtcod.BKGND_NONE, libtcod.CENTER,
                'Spectres: ' +str(spectrenewb_kill))  
        if greywolf_kill > 0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-4, libtcod.BKGND_NONE, libtcod.CENTER,
                'Grey Wolves: ' + str(greywolf_kill)) 
        if snaga_kill > 0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-3, libtcod.BKGND_NONE, libtcod.CENTER,
                'Snagas: ' + str(snaga_kill))       
        if warg_kill >0 :        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-2, libtcod.BKGND_NONE, libtcod.CENTER,
                'Wargs: ' + str(warg_kill))       
        if spectre_kill >0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-1, libtcod.BKGND_NONE, libtcod.CENTER,
                'Royal Spectres: ' +str(spectre_kill))       
        if snuffler_kill >0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
                'Snufflers: ' +str(snuffler_kill))       
        if wargleader_kill >0:        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2+1, libtcod.BKGND_NONE, libtcod.CENTER,
                'Warg Leaders: ' + str(wargleader_kill))                   
            
            
        libtcod.console_set_default_foreground(0, libtcod.white)  
        
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT - 3, libtcod.BKGND_NONE, libtcod.CENTER,
            'K) to return to Game')                   
            
        choice = page_menu('', [''], 24)
                        
        if choice == 0:
            break
        else:
            break
                        
def create_player():
    global TORCH_RADIUS
    global THROW_DAMAGE
    global regen_rate , REGEN_LEVEL
    global power_bonus 
    global throwing_skill
    global weight_cap
    global tract_power ,tract_defence
    global take_games
    global turn_step , turn_step_level    
    
    turn_step = 0 
    turn_step_level = 0
    
    tract_power = 1
    tract_defence =1    
    weight_cap = 10
    regen_rate = 30
    THROW_DAMAGE = 2
    skill_int  = 2
    
    screen_track = 0
    
    create_a_height = 14
    create_a_cursor = -10
    cost_lane = 10
    
    colour_factor = libtcod.peach

    img2 = libtcod.image_load('Coloured_helmet2.png') 
    new_game()  
    
    while not libtcod.console_is_window_closed():
        #show the background image, at twice the regular console resolution
        libtcod.image_blit_2x(img2, 0, 0, 0)   
        
        if screen_track < 0:
            screen_track = 9
        if screen_track > 9:
            screen_track = 0          
        
        #show the game's title, and some credits!
        libtcod.console_set_default_foreground(0, libtcod.light_azure)
        
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-15, libtcod.BKGND_NONE, libtcod.CENTER,
            'Create your dwarven hero!') 
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-13, libtcod.BKGND_NONE, libtcod.CENTER,
            'Name: ' + name_fetch.got_name())     
        libtcod.console_print_ex(0, SCREEN_WIDTH/2 , SCREEN_HEIGHT/2-12, libtcod.BKGND_NONE, libtcod.CENTER,
            'Title: ' + name_fetch.got_h())               
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-11, libtcod.BKGND_NONE, libtcod.CENTER,
            'Profession: ' + name_fetch.got_m())             
            
        if skill_int ==0:    
            libtcod.console_set_default_foreground(0, libtcod.red)            
        
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-7, libtcod.BKGND_NONE, libtcod.CENTER, 'Skill Points = ' + str(skill_int)) 
        libtcod.console_set_default_foreground(0, libtcod.light_azure)  
#######################################################COST AND BUYING########################            
        
        libtcod.console_set_default_foreground(0, libtcod.white)        
        if screen_track == 0:
            libtcod.console_set_default_foreground(0, colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2 + create_a_cursor, create_a_height+8, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+8, libtcod.BKGND_NONE, libtcod.CENTER,
            'Add Hit Points')
        libtcod.console_print_ex(0, SCREEN_WIDTH/2 + cost_lane, create_a_height+8, libtcod.BKGND_NONE, libtcod.CENTER,
            '1p')            
            
        libtcod.console_set_default_foreground(0, libtcod.white)        
        if screen_track == 1:
            libtcod.console_set_default_foreground(0, colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+ create_a_cursor, create_a_height+9, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+9, libtcod.BKGND_NONE, libtcod.CENTER,
            'Better Offence')
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+ cost_lane, create_a_height+9, libtcod.BKGND_NONE, libtcod.CENTER,
            '1p')            

        libtcod.console_set_default_foreground(0, libtcod.white)        
        if screen_track == 2:
            libtcod.console_set_default_foreground(0, colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+ create_a_cursor, create_a_height+10, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+10, libtcod.BKGND_NONE, libtcod.CENTER,
            'Better Defence')
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+ cost_lane, create_a_height+10, libtcod.BKGND_NONE, libtcod.CENTER,
            '1p')            

        libtcod.console_set_default_foreground(0, libtcod.white)        
        if screen_track == 3:
            libtcod.console_set_default_foreground(0,colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+ create_a_cursor, create_a_height+11, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+11, libtcod.BKGND_NONE, libtcod.CENTER,
            'Adept Throwing')
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+ cost_lane, create_a_height+11, libtcod.BKGND_NONE, libtcod.CENTER,
            '1p')            

        libtcod.console_set_default_foreground(0, libtcod.white)        
        if screen_track == 4:
            libtcod.console_set_default_foreground(0, colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+ create_a_cursor, create_a_height+12, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+12, libtcod.BKGND_NONE, libtcod.CENTER,
            'Better Endurance')
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+ cost_lane, create_a_height+12, libtcod.BKGND_NONE, libtcod.CENTER,
            '1p')            

        libtcod.console_set_default_foreground(0, libtcod.white)        
        if screen_track == 5:
            libtcod.console_set_default_foreground(0,colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+ create_a_cursor, create_a_height+13, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+13, libtcod.BKGND_NONE, libtcod.CENTER,
            'Carry More')
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+ cost_lane, create_a_height+13, libtcod.BKGND_NONE, libtcod.CENTER,
            '1p')            

      

        libtcod.console_set_default_foreground(0, libtcod.white)        
        if screen_track == 6:
            libtcod.console_set_default_foreground(0,colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+ create_a_cursor, create_a_height+14, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+14, libtcod.BKGND_NONE, libtcod.CENTER,
            'Lucky')  
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+ cost_lane, create_a_height+14, libtcod.BKGND_NONE, libtcod.CENTER,
            '2p')            

        libtcod.console_set_default_foreground(0, libtcod.white)               
        if screen_track == 7:
            libtcod.console_set_default_foreground(0,colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+ create_a_cursor, create_a_height+16, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+16, libtcod.BKGND_NONE, libtcod.CENTER,
            'Reset Attributes')              
            
        libtcod.console_set_default_foreground(0, libtcod.white)   
        if screen_track == 8:
            libtcod.console_set_default_foreground(0,colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+ create_a_cursor, create_a_height+17, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+17, libtcod.BKGND_NONE, libtcod.CENTER,
            'Back to Main Menu')   
            
        libtcod.console_set_default_foreground(0, libtcod.white)   
        if screen_track == 9:
            libtcod.console_set_default_foreground(0,colour_factor)          
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+ create_a_cursor, create_a_height+18, libtcod.BKGND_NONE, libtcod.CENTER, 
                '@' )
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, create_a_height+18, libtcod.BKGND_NONE, libtcod.CENTER,
            '--> Ready <--')               
            
            
#######################################################SKILL SHOW LIST#########################    
        libtcod.console_set_default_foreground(0, libtcod.light_azure) 
        if int(player.fighter.max_hp) > 5:
            libtcod.console_set_default_foreground(0, libtcod.green)          
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2+8, libtcod.BKGND_NONE, libtcod.CENTER, 'Max Hit Points = ' + str(player.fighter.max_hp ))
        libtcod.console_set_default_foreground(0, libtcod.light_azure)    
        
        if int(player.fighter.power) > 1:
            libtcod.console_set_default_foreground(0, libtcod.green)            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2+9, libtcod.BKGND_NONE, libtcod.CENTER, 'Offence = ' + str(player.fighter.power))
        libtcod.console_set_default_foreground(0, libtcod.light_azure)                    
        
        if int(player.fighter.defence) > 1:
            libtcod.console_set_default_foreground(0, libtcod.green)            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2+10, libtcod.BKGND_NONE, libtcod.CENTER,'Defence = ' + str(player.fighter.defence))
        libtcod.console_set_default_foreground(0, libtcod.light_azure) 
        
        if int(THROW_DAMAGE - 1) > 1:
            libtcod.console_set_default_foreground(0, libtcod.green)            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2+11, libtcod.BKGND_NONE, libtcod.CENTER, 'Throwing Proficiency = ' + str(THROW_DAMAGE - 1))  
        libtcod.console_set_default_foreground(0, libtcod.light_azure) 
        
        regen_level()            
        if int(REGEN_LEVEL) > 1:
            libtcod.console_set_default_foreground(0, libtcod.green)                
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2+12, libtcod.BKGND_NONE, libtcod.CENTER, 'Level of Endurance = ' + str(REGEN_LEVEL))       
        libtcod.console_set_default_foreground(0, libtcod.light_azure) 
        
        if int(weight_cap) > 10:
            libtcod.console_set_default_foreground(0, libtcod.green)            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2+13, libtcod.BKGND_NONE, libtcod.CENTER, 'Max Carry Weight = ' + str(weight_cap) + 'kg')         
        libtcod.console_set_default_foreground(0, libtcod.light_azure) 
                
        if int(player.fighter.luck) > 0:
            libtcod.console_set_default_foreground(0, libtcod.green)            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2+14, libtcod.BKGND_NONE, libtcod.CENTER, 'Luck = ' + str(player.fighter.luck))           
        libtcod.console_set_default_foreground(0, libtcod.light_azure) 
        
        libtcod.console_flush()
        key = libtcod.console_wait_for_keypress(True)   
            
        if libtcod.console_is_key_pressed(libtcod.KEY_UP):
            screen_track -= 1
        elif libtcod.console_is_key_pressed(libtcod.KEY_DOWN):
            screen_track += 1
        print screen_track        
                        
        if key.vk == libtcod.KEY_ENTER and screen_track == 0:
            if 0 < skill_int:
                player.fighter.base_max_hp += 5
                player.fighter.hp += 0
                skill_int -=1 
                
        if key.vk == libtcod.KEY_ENTER and screen_track == 1:
            if 0 < skill_int:
                    player.fighter.base_power += 1
                    
                    tract_power += 1
                    skill_int -=1         
                
        if key.vk == libtcod.KEY_ENTER and screen_track == 2:
            if 0 < skill_int:
                player.fighter.base_defence += 1
                
                tract_defence += 1
                skill_int -=1

        if key.vk == libtcod.KEY_ENTER and screen_track == 3:
            if 0 < skill_int:
                THROW_DAMAGE +=1
                skill_int -=1                       
                
        if key.vk == libtcod.KEY_ENTER and screen_track == 4:
            if 0 < skill_int:
                regen_rate -=1
                skill_int -=1          

        if key.vk == libtcod.KEY_ENTER and screen_track == 5:
            if 0 < skill_int:
                weight_cap += 5
                skill_int -=1                

        if key.vk == libtcod.KEY_ENTER and screen_track == 6:
            if 1 < skill_int:
                player.fighter.base_luck += 1
                skill_int -= 2                                                                        

        if key.vk == libtcod.KEY_ENTER and screen_track == 7:
            player.fighter.base_max_hp = 5
            player.fighter.hp= 5
            player.fighter.base_power = 1 
            player.fighter.base_defence =1 
            regen_rate = 30
            weight_cap = 10
            player.fighter.base_luck = 0
            THROW_DAMAGE = 2
            skill_int  = 2
            
            tract_power = 1
            tract_defence = 1
            
        if key.vk == libtcod.KEY_ENTER and screen_track == 8:
            break
          
        if key.vk == libtcod.KEY_ENTER and screen_track == 9:
            take_games +=1 
            play_game()
            
            break
            
def options():
    global option_hint , option_fullscreen

    img = libtcod.image_load('Blank.png')
    screen_track = 0
    
    Lane_one = SCREEN_WIDTH/3
    Lane_two = 2*(SCREEN_WIDTH/3)
    Lane_three = Lane_two + 4
    

    
    while not libtcod.console_is_window_closed():
        #show the background image, at twice the regular console resolution
        libtcod.image_blit_2x(img, 0, 0, 0)    
        if screen_track < 0:
            screen_track = 2
        if screen_track > 2:
            screen_track = 0
            
        libtcod.console_set_default_foreground(0, libtcod.white)            
        if screen_track == 0:
            libtcod.console_set_default_foreground(0, libtcod.light_azure)        
            libtcod.console_print_ex(0, Lane_one, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ Hints @') 
        libtcod.console_print_ex(0, Lane_one, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Hints')    
        libtcod.console_set_default_foreground(0, libtcod.gray)            
        if option_hint == 0:
            libtcod.console_set_default_foreground(0, libtcod.green)        
        libtcod.console_print_ex(0, Lane_two, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
            'On') 
        libtcod.console_set_default_foreground(0, libtcod.gray)              
        if option_hint == 1:
            libtcod.console_set_default_foreground(0, libtcod.red)                 
        libtcod.console_print_ex(0, Lane_three, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Off')                
            
        libtcod.console_set_default_foreground(0, libtcod.white) 
        if screen_track == 1:
            libtcod.console_set_default_foreground(0, libtcod.light_azure)        
            libtcod.console_print_ex(0, Lane_one, SCREEN_HEIGHT/2 +1, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ Full Screen @') 
        libtcod.console_print_ex(0, Lane_one, SCREEN_HEIGHT/2 +1, libtcod.BKGND_NONE, libtcod.CENTER,
            'Full Screen')            
        libtcod.console_set_default_foreground(0, libtcod.gray)                    
        if option_fullscreen == 0:
            libtcod.console_set_default_foreground(0, libtcod.green)        
        libtcod.console_print_ex(0, Lane_two, SCREEN_HEIGHT/2+1, libtcod.BKGND_NONE, libtcod.CENTER,
            'On') 
        libtcod.console_set_default_foreground(0, libtcod.gray)              
        if option_fullscreen == 1:
            libtcod.console_set_default_foreground(0, libtcod.red)                 
        libtcod.console_print_ex(0, Lane_three, SCREEN_HEIGHT/2+1, libtcod.BKGND_NONE, libtcod.CENTER,
            'Off')               
            
        libtcod.console_set_default_foreground(0, libtcod.white) 
        if screen_track == 2:
            libtcod.console_set_default_foreground(0, libtcod.light_azure)        
            libtcod.console_print_ex(0, Lane_one, SCREEN_HEIGHT/2 +2, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ Back @') 
        libtcod.console_print_ex(0, Lane_one, SCREEN_HEIGHT/2 +2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Back')                 
        
        libtcod.console_flush()
        key = libtcod.console_wait_for_keypress(True)   
            
        if libtcod.console_is_key_pressed(libtcod.KEY_UP):
            screen_track -= 1
        elif libtcod.console_is_key_pressed(libtcod.KEY_DOWN):
            screen_track += 1              

        if screen_track == 0 and key.vk == libtcod.KEY_ENTER:
            if option_hint == 0:
                option_hint = 1
            else:
                option_hint = 0
                
        if screen_track == 1 and key.vk == libtcod.KEY_ENTER:
            if option_fullscreen == 0:
                option_fullscreen = 1
            else:
                option_fullscreen = 0                
            libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())   
            
        if screen_track == 2 and key.vk == libtcod.KEY_ENTER:
            screen_track = 3
            break                      

            
            
    
    
            
def stat_screen():
    global screen_track , small_steps , big_steps , take_steps_n , take_floor , take_games
    
    img = libtcod.image_load('Blank.png')
    screen_track = 0
    stat_lane = 16    
    number_lane = 32
    number_lane_2 = 48
    number_lane_3 = 64
    number_lane_4 = 80    
    
    steps_height = SCREEN_HEIGHT/2-20
    floors_height = SCREEN_HEIGHT/2-19
    games_height = SCREEN_HEIGHT/2-18
    
    while not libtcod.console_is_window_closed():
        #show the background image, at twice the regular console resolution
        libtcod.image_blit_2x(img, 0, 0, 0)    
        if screen_track < 0:
            screen_track = 1
        if screen_track > 1:
            screen_track = 0
        
        libtcod.console_set_default_foreground(0, libtcod.azure)
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-25, libtcod.BKGND_NONE, libtcod.CENTER,
            'Your Statistics')    
            
        libtcod.console_set_default_foreground(0, libtcod.white) 
        libtcod.console_print_ex(0, number_lane, SCREEN_HEIGHT/2-21, libtcod.BKGND_NONE, libtcod.CENTER,
            'Least:')   
        libtcod.console_print_ex(0, number_lane_2, SCREEN_HEIGHT/2-21, libtcod.BKGND_NONE, libtcod.CENTER,
            'Average:')   
        libtcod.console_print_ex(0, number_lane_3, SCREEN_HEIGHT/2-21, libtcod.BKGND_NONE, libtcod.CENTER,
            'Most:')     
        libtcod.console_print_ex(0, number_lane_4, SCREEN_HEIGHT/2-21, libtcod.BKGND_NONE, libtcod.CENTER,
            'Total:')    
            
        libtcod.console_print_ex(0, stat_lane, steps_height, libtcod.BKGND_NONE, libtcod.CENTER,
            'Steps:')   
        check_step = small_steps
        if check_step == 10000:
                libtcod.console_print_ex(0, number_lane, steps_height, libtcod.BKGND_NONE, libtcod.CENTER,
                    '0' )   
        else:            
            libtcod.console_print_ex(0, number_lane, steps_height, libtcod.BKGND_NONE, libtcod.CENTER,
                str(small_steps) )        

        check_take_floor = take_floor
        if check_take_floor == 0:
            libtcod.console_print_ex(0, number_lane_2, steps_height, libtcod.BKGND_NONE, libtcod.CENTER,
                '0' )           
        else:
            libtcod.console_print_ex(0, number_lane_2, steps_height, libtcod.BKGND_NONE, libtcod.CENTER,
                str(take_steps/ (take_floor/2)) )                
        libtcod.console_print_ex(0, number_lane_3, steps_height, libtcod.BKGND_NONE, libtcod.CENTER,
            str(big_steps) )                
        libtcod.console_print_ex(0, number_lane_4, steps_height, libtcod.BKGND_NONE, libtcod.CENTER,
            str(take_steps + take_steps_n) )      

            
        libtcod.console_print_ex(0, stat_lane, floors_height, libtcod.BKGND_NONE, libtcod.CENTER,
            'Floors:') 
        check_take_games = take_games
        if check_take_games == 0:
            libtcod.console_print_ex(0, number_lane_2, floors_height, libtcod.BKGND_NONE, libtcod.CENTER,
                '0' )           
        else:    
            libtcod.console_print_ex(0, number_lane_2, floors_height, libtcod.BKGND_NONE, libtcod.CENTER,
                str((take_floor/2)/take_games) )              
        libtcod.console_print_ex(0, number_lane_4, floors_height, libtcod.BKGND_NONE, libtcod.CENTER,
            str(take_floor/2) )           

        libtcod.console_print_ex(0, stat_lane, games_height, libtcod.BKGND_NONE, libtcod.CENTER,
            'Games:') 
        libtcod.console_print_ex(0, number_lane_4, games_height, libtcod.BKGND_NONE, libtcod.CENTER,
            str(take_games) )                   
            
        libtcod.console_set_default_foreground(0, libtcod.white)            
        if screen_track == 0:
            libtcod.console_set_default_foreground(0, libtcod.light_azure)        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT - 5, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ Back @') 
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT -5, libtcod.BKGND_NONE, libtcod.CENTER,
            'Back')    
            
        libtcod.console_set_default_foreground(0, libtcod.white) 
        if screen_track == 1:
            libtcod.console_set_default_foreground(0, libtcod.light_azure)        
            libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT-6, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ Reset Statistics @') 
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT -6, libtcod.BKGND_NONE, libtcod.CENTER,
            'Reset Statistics')             
        
        libtcod.console_flush()
        key = libtcod.console_wait_for_keypress(True)   
            
        if libtcod.console_is_key_pressed(libtcod.KEY_UP):
            screen_track -= 1
        elif libtcod.console_is_key_pressed(libtcod.KEY_DOWN):
            screen_track += 1              

        if screen_track == 0 and key.vk == libtcod.KEY_ENTER:
            screen_track = 3
            break            
        if screen_track == 1 and key.vk == libtcod.KEY_ENTER:
            wipe_stat()
            
def wipe_stat():  
    global screen_track , small_steps , big_steps , take_steps_n , take_steps , take_floor , take_games
    
    screen_track = 0
    img = libtcod.image_load('Blank.png')    
    
    while not libtcod.console_is_window_closed():  
        libtcod.image_blit_2x(img, 0, 0, 0)       

        if screen_track < 0:
            screen_track = 0
        if screen_track > 1:
            screen_track = 1            
                
        libtcod.console_set_default_foreground(0, libtcod.green) 
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-4, libtcod.BKGND_NONE, libtcod.CENTER,
            'Resetting Stats! Are You Sure?')   
                    
        libtcod.console_set_default_foreground(0, libtcod.white) 
        if screen_track == 1:
            libtcod.console_set_default_foreground(0, libtcod.red)            
            libtcod.console_print_ex(0, SCREEN_WIDTH/2-5, SCREEN_HEIGHT/2 -2, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ Yes! @')              
        libtcod.console_print_ex(0, SCREEN_WIDTH/2-5 , SCREEN_HEIGHT/2 - 2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Yes!')   
                    
        libtcod.console_set_default_foreground(0, libtcod.white) 
        if screen_track == 0:
            libtcod.console_set_default_foreground(0, libtcod.light_azure)            
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+5, SCREEN_HEIGHT/2 -2, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ No! @')              
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+5, SCREEN_HEIGHT/2 -2, libtcod.BKGND_NONE, libtcod.CENTER,
            'No!')            
            
        libtcod.console_flush()
        key = libtcod.console_wait_for_keypress(True)              

        if libtcod.console_is_key_pressed(libtcod.KEY_LEFT):
            screen_track += 1
        elif libtcod.console_is_key_pressed(libtcod.KEY_RIGHT):
            screen_track -= 1   
            
        if screen_track == 0 and key.vk == libtcod.KEY_ENTER:
            break            
        if screen_track == 1 and key.vk == libtcod.KEY_ENTER:            
            take_steps = 0
            turn_count = 0
            take_steps_n = 0
            take_floor = 0
            take_games = 0
            small_steps = 10000
            big_steps = 0            
            save_stat()
            break
            
def exit_game():  
    global screen_track , small_steps , big_steps , take_steps_n , take_steps , take_floor , take_games
    
    screen_track = 0
    img = libtcod.image_load('Blank.png')    
    
    while not libtcod.console_is_window_closed():  
        libtcod.image_blit_2x(img, 0, 0, 0)       

        if screen_track < 0:
            screen_track = 0
        if screen_track > 1:
            screen_track = 1            
                
        libtcod.console_set_default_foreground(0, libtcod.white) 
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-4, libtcod.BKGND_NONE, libtcod.CENTER,
            'Are you leaving us?')   
                    
        libtcod.console_set_default_foreground(0, libtcod.white) 
        if screen_track == 1:
            libtcod.console_set_default_foreground(0, libtcod.red)            
            libtcod.console_print_ex(0, SCREEN_WIDTH/2-5, SCREEN_HEIGHT/2 -2, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ Yes! @')              
        libtcod.console_print_ex(0, SCREEN_WIDTH/2-5 , SCREEN_HEIGHT/2 - 2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Yes!')   
                    
        libtcod.console_set_default_foreground(0, libtcod.white) 
        if screen_track == 0:
            libtcod.console_set_default_foreground(0, libtcod.light_azure)            
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+5, SCREEN_HEIGHT/2 -2, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ No! @')              
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+5, SCREEN_HEIGHT/2 -2, libtcod.BKGND_NONE, libtcod.CENTER,
            'No!')            
            
        libtcod.console_flush()
        key = libtcod.console_wait_for_keypress(True)              

        if libtcod.console_is_key_pressed(libtcod.KEY_LEFT):
            screen_track += 1
        elif libtcod.console_is_key_pressed(libtcod.KEY_RIGHT):
            screen_track -= 1   
            
        if screen_track == 0 and key.vk == libtcod.KEY_ENTER:
            screen_track = 4
            break            
        if screen_track == 1 and key.vk == libtcod.KEY_ENTER:     
            exit()
               

def main_menu():
    global screen_track
    img = libtcod.image_load('Coloured_helmet_shade_and_colours.png')
    
    screen_track = 0
 
    while not libtcod.console_is_window_closed():
        #show the background image, at twice the regular console resolution
        libtcod.image_blit_2x(img, 0, 0, 0)
    #    libtcod.console_flush()
      #  key = libtcod.console_wait_for_keypress(True)        
        
        if screen_track < 0:
            screen_track = 4
        if screen_track > 4:
            screen_track = 0        
        
        #show the game's title, and some credits!
        libtcod.console_set_default_foreground(0, libtcod.lighter_azure)
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-25, libtcod.BKGND_NONE, libtcod.CENTER,
            '{THE LORE OF DWAVES}')
        libtcod.console_print_ex(0, SCREEN_WIDTH/2, SCREEN_HEIGHT/2-24, libtcod.BKGND_NONE, libtcod.CENTER,
            'By Fervour(mm11wils)')
            
        libtcod.console_set_default_foreground(0, libtcod.white)    
        
        if screen_track == 0:
            libtcod.console_set_default_foreground(0, libtcod.turquoise)  
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2-1, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ New Game @')
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2-1, libtcod.BKGND_NONE, libtcod.CENTER,
            'New Game')
        
        libtcod.console_set_default_foreground(0, libtcod.white)  
        if screen_track == 1:
            libtcod.console_set_default_foreground(0, libtcod.turquoise)  
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ Continue @')            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Continue')          

        libtcod.console_set_default_foreground(0, libtcod.white)  
        if screen_track == 2:
            libtcod.console_set_default_foreground(0, libtcod.turquoise)  
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2+1, libtcod.BKGND_NONE, libtcod.CENTER,
                '@  Options @')            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2+1, libtcod.BKGND_NONE, libtcod.CENTER,
            'Options') 

        libtcod.console_set_default_foreground(0, libtcod.white)  
        if screen_track == 3:
            libtcod.console_set_default_foreground(0, libtcod.turquoise)  
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2+2, libtcod.BKGND_NONE, libtcod.CENTER,
                '@ Statistics @')            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2+2, libtcod.BKGND_NONE, libtcod.CENTER,
            'Statistics')             
            
        libtcod.console_set_default_foreground(0, libtcod.white)  
        if screen_track == 4:
            libtcod.console_set_default_foreground(0, libtcod.turquoise)  
            libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2+3, libtcod.BKGND_NONE, libtcod.CENTER,
                '@   Quit   @')            
        libtcod.console_print_ex(0, SCREEN_WIDTH/2+35, SCREEN_HEIGHT/2+3, libtcod.BKGND_NONE, libtcod.CENTER,
            'Quit')     
            
        libtcod.console_flush()
        key = libtcod.console_wait_for_keypress(True)   
            
        if libtcod.console_is_key_pressed(libtcod.KEY_UP):
            screen_track -= 1
        elif libtcod.console_is_key_pressed(libtcod.KEY_DOWN):
            screen_track += 1
        print screen_track
        
        if key.vk == libtcod.KEY_ENTER and screen_track == 0:
            create_player() 
        if key.vk == libtcod.KEY_ENTER and screen_track == 1:    
            try:
                load_game()
                name_fetch.load_game()
                Dungeon_story.load_game()
            except:
                msgbox('\n No saved game to load.\n', 24)
                continue
            play_game() 
        if key.vk == libtcod.KEY_ENTER and screen_track == 2:  
            options()
        if key.vk == libtcod.KEY_ENTER and screen_track == 3: 
         #   try:
            load_stat()
            stat_screen()
            #except:
              #  msgbox('\n No saved game to load.\n', 24)
                #continue

        if key.vk == libtcod.KEY_ENTER and screen_track == 4:
            exit_game()
 
libtcod.console_set_custom_font('arial10x1023.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)
libtcod.console_init_root(SCREEN_WIDTH, SCREEN_HEIGHT, 'THE LORE OF DWAVES', False)
libtcod.sys_set_fps(LIMIT_FPS)
con = libtcod.console_new(MAP_WIDTH, MAP_HEIGHT) #was at screen w/h at one stage
panel = libtcod.console_new(SCREEN_WIDTH-20, PANEL_HEIGHT)
panel_right = libtcod.console_new(PANEL_RIGHT_W, SCREEN_HEIGHT)
print libtcod.sys_get_current_resolution()

main_menu()
