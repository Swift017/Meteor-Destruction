```lua
-- Swift

-- Services
local ReplicatedStorage = game:GetService("ReplicatedStorage");
local RunService = game:GetService("RunService");
local Rex = require(ReplicatedStorage.Rex);

local RayService = Rex:GetModule("RayService")
local Math = Rex.Math
local HitboxService = Rex:GetModule("HitboxService")
local VFX = ReplicatedStorage.Remotes.VFXRemote;
local Mouse = ReplicatedStorage.Remotes.GetMouseHit;
local TaskScheduler = Rex:GetModule("TaskScheduler");

-- Code
return function(Player, MouseHit, SkillData)
	Rex:Write("States", {Player.Name, "Aiming"}, true);
	local DamageModule = Rex:GetModule("Damage");	

	local Character = Player.Character;
	local MousePosition = MouseHit.Position;
	
	local SpawnPoint = CFrame.lookAt(Character.PrimaryPart.Position + Vector3.new(0,200,0), MouseHit.Position);
	local Size = ReplicatedStorage.Assets.Esper.Swift.Meteor.Size.X * 1.5;
	local Velocity = 200;
	local Lifetime = 10;
	local Damage = SkillData.Damage;
	
	VFX:FireAllClients(script.Name, {
		Character = Player.Character;
		MouseHit = MouseHit;
		Size = Size;
		SpawnPoint = SpawnPoint;
		Velocity = Velocity;
		Lifetime = Lifetime;
	})	
	
	local Points = HitboxService:GetSquarePoints(SpawnPoint, Size, Size);
	HitboxService:CastProjectileHitbox({ -- Everything is required
		Points = Points, -- Array Of CFrames
		Direction = (MouseHit.Position - SpawnPoint.Position).Unit, -- Direction
		Velocity = Velocity, -- Velocity Of Projectile
		Lifetime = Lifetime, -- Total Duration 
		Iterations = 50, -- Amount of times it's splitted,
		Visualize = false, -- Visualizes the hitbox using RayService
		Function = function(RaycastResult) -- Callback
			-- | AOE DMG
			local ValidEntities = HitboxService:GetEntitiesFromPoint(RaycastResult.Position, game.Workspace.Entities:GetChildren(), {[Player.Character] = true}, Size)
			for i = 1, #ValidEntities do
				local Entity = ValidEntities[i];
				DamageModule.InflictDamage(Player, Entity.Humanoid, Damage);
			end
		end,
		Ignore = {Player.Character, game.Workspace.Visuals} -- Array Of Objects To be Ignored
	})
	
	TaskScheduler:AddItem(0.5, function()
		Rex:Write("States", {Player.Name, "Aiming"}, false);
	end)
	wait(0.5);
end
```
