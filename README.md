package me.kalo121;

import java.util.ArrayList;
import org.bukkit.Chunk;
import org.bukkit.World;
import org.bukkit.entity.Arrow;
import org.bukkit.entity.Egg;
import org.bukkit.entity.Entity;
import org.bukkit.entity.Fish;
import org.bukkit.entity.LivingEntity;
import org.bukkit.entity.Player;
import org.bukkit.entity.Snowball;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.block.Action;
import org.bukkit.event.block.BlockBreakEvent;
import org.bukkit.event.block.BlockPlaceEvent;
import org.bukkit.event.entity.EntityDamageByEntityEvent;
import org.bukkit.event.entity.EntityDamageEvent;
import org.bukkit.event.entity.EntityShootBowEvent;
import org.bukkit.event.player.PlayerBucketEmptyEvent;
import org.bukkit.event.player.PlayerBucketFillEvent;
import org.bukkit.event.player.PlayerInteractEvent;
import org.bukkit.event.player.PlayerJoinEvent;
import org.bukkit.event.player.PlayerPickupItemEvent;

public class EListener
  implements Listener
{
  private ArrayList<String> fallingPlayers = new ArrayList<String>();

  @EventHandler
  public void onEntityDamage(EntityDamageEvent event)
  {
    Entity eDamaged = event.getEntity();
    Player pDamaged = null;
    Player pAttacker = null;
    LivingEntity livingEntity = null;
    Arrow shotarrow = null;
    EntityDamageByEntityEvent ev = null;
    Entity damager = null;
    LivingEntity shooter = null;
    Boolean divebombignore = Boolean.valueOf(false);

    if ((eDamaged instanceof Player))
      pDamaged = (Player)eDamaged;
    else if ((eDamaged instanceof LivingEntity)) {
      livingEntity = (LivingEntity)eDamaged;
    }

    if ((event instanceof EntityDamageByEntityEvent)) {
      ev = (EntityDamageByEntityEvent)event;
      damager = ev.getDamager();
      if ((damager instanceof Player)) {
        pAttacker = (Player)damager;
      } else if ((damager instanceof Arrow)) {
        shotarrow = (Arrow)damager;

        shooter = (LivingEntity) shotarrow.getShooter();
        if ((shooter instanceof Player)) {
          pAttacker = (Player)shooter;
        }
      }
      else if ((damager instanceof Egg)) {
        divebombignore = Boolean.valueOf(true);
      } else if ((damager instanceof Snowball)) {
        divebombignore = Boolean.valueOf(true);
      } else if ((damager instanceof Fish)) {
        divebombignore = Boolean.valueOf(true);
      }

    }

    if ((pDamaged != null) && (pAttacker != null) && 
      (pAttacker.getAllowFlight()) && (!pDamaged.getAllowFlight()) && (pAttacker.getItemInHand().getTypeId() != 261) && 
      (Settings.pvpBlocked()) && (!pAttacker.hasPermission("flydisable.exempt.pvp"))) {
      String warningMsg = Settings.getPVPWarningMsg();
      String disableMsg = Settings.getPVPDisableMsg();
      if ((warningMsg != "") && (warningMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(pAttacker, "PVPWarning", warningMsg);
      }

      if ((disableMsg != "") && (disableMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(pAttacker, "PVPDisable", disableMsg);
      }

      pAttacker.updateInventory();
      event.setCancelled(true);
      return;
    }

    if ((pDamaged != null) && (pDamaged.getAllowFlight()) && (Settings.flyDisabledOnHit()) && (!pDamaged.hasPermission("flydisable.exempt.divebomb"))) {
      if ((!Settings.mobDisableFly()) && (pAttacker == null))
        return;
      if (divebombignore.booleanValue())
        return;
      if ((pAttacker != null) && (pAttacker.getAllowFlight())) {
        return;
      }
      String message = Settings.getDiveBombMsg();
      pDamaged.setAllowFlight(false);
      this.fallingPlayers.add(pDamaged.getName());
      MsgThread.getInstance().sendMsg(pDamaged, "DiveBomb", message);
      return;
    }

    if ((pAttacker != null) && (pDamaged != null) && (pAttacker.getAllowFlight()) && (!pDamaged.getAllowFlight()) && 
      (shotarrow != null) && (!shotarrow.isDead()) && (pAttacker.getItemInHand().getTypeId() == 261) && (Settings.bowDamagePlayerBlocked()) && (!pAttacker.hasPermission("flydisable.exempt.bowdamageplayer"))) {
      String warningMsg = Settings.getBowDamagePlayerWarningMsg();
      String disableMsg = Settings.getBowDamagePlayerDisableMsg();

      if ((warningMsg != "") && (warningMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(pAttacker, "BowDamagePlayerWarning", warningMsg);
      }

      if ((disableMsg != "") && (disableMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(pAttacker, "BowDamagePlayerDisable", disableMsg);
      }

      pAttacker.updateInventory();
      event.setCancelled(true);
      return;
    }

    if ((pAttacker != null) && (livingEntity != null) && (pAttacker.getAllowFlight()) && 
      (shotarrow != null) && 
      (!shotarrow.isDead()) && 
      (pAttacker.getItemInHand().getTypeId() == 261) && 
      (Settings.bowDamageMobBlocked()) && 
      (!pAttacker.hasPermission("flydisable.exempt.bowdamagemob")))
    {
      String warningMsg = Settings.getBowDamageMobWarningMsg();
      String disableMsg = Settings.getBowDamageMobDisableMsg();

      if ((warningMsg != "") && (warningMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(pAttacker, "BowDamageMobWarning", warningMsg);
      }

      if ((disableMsg != "") && (disableMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(pAttacker, "BowDamageMobDisable", disableMsg);
      }

      shotarrow.setBounce(false);
      shotarrow.remove();

      pAttacker.updateInventory();
      event.setCancelled(true);
      return;
    }
  }

  @EventHandler
  public void onPlayerInteract(PlayerInteractEvent event)
  {
    if ((event.getAction() == Action.RIGHT_CLICK_AIR) || (event.getAction() == Action.RIGHT_CLICK_BLOCK)) {
      Player player = event.getPlayer();

      if ((player.getItemInHand().getTypeId() == 373) && (player.getAllowFlight()) && 
        (Settings.potionsBlocked()) && (!player.hasPermission("flydisable.exempt.potions"))) {
        String warningMsg = Settings.getPotionWarningMsg();
        String disableMsg = Settings.getPotionDisableMsg();
        if ((warningMsg != "") && (warningMsg.length() > 0)) {
          MsgThread.getInstance().sendMsg(player, "PotionWarning", warningMsg);
        }

        if ((disableMsg != "") && (disableMsg.length() > 0)) {
          MsgThread.getInstance().sendMsg(player, "PotionDisable", disableMsg);
        }

        player.updateInventory();
        event.setCancelled(true);
      }
    }
  }

  @EventHandler
  public void onEntityShootBow(EntityShootBowEvent event)
  {
    if ((event.getEntity() instanceof Player))
    {
      Player player = (Player)event.getEntity();
      if ((player.getAllowFlight()) && 
        (Settings.bowFireBlocked()) && (!player.hasPermission("flydisable.exempt.bowfire"))) {
        String warningMsg = Settings.getBowFireWarningMsg();
        String disableMsg = Settings.getBowFireDisableMsg();
        if ((warningMsg != "") && (warningMsg.length() > 0)) {
          MsgThread.getInstance().sendMsg(player, "BowFireWarning", warningMsg);
        }

        if ((disableMsg != "") && (disableMsg.length() > 0)) {
          MsgThread.getInstance().sendMsg(player, "BowFireDisable", disableMsg);
        }

        player.updateInventory();
        event.setCancelled(true);
      }
    }
  }

  @EventHandler
  public void onBlockBreak(BlockBreakEvent event)
  {
    if ((event.getPlayer().getAllowFlight()) && (Settings.blockBreakBlocked()) && (!event.getPlayer().hasPermission("flydisable.exempt.blockbreak"))) {
      String warningMsg = Settings.getBlockBreakWarningMsg();
      String disableMsg = Settings.getBlockBreakDisableMsg();
      if ((warningMsg != "") && (warningMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "BlockBreakWarning", warningMsg);
      }

      if ((disableMsg != "") && (disableMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "BlockBreakDisable", disableMsg);
      }

      event.getPlayer().updateInventory();
      event.setCancelled(true);
    }
  }

  @EventHandler
  public void onBlockPlace(BlockPlaceEvent event)
  {
    if ((event.getPlayer().getAllowFlight()) && (Settings.blockPlaceBlocked()) && (!event.getPlayer().hasPermission("flydisable.exempt.blockplace"))) {
      String warningMsg = Settings.getBlockPlaceWarningMsg();
      String disableMsg = Settings.getBlockPlaceDisableMsg();
      if ((warningMsg != "") && (warningMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "BlockPlaceWarning", warningMsg);
      }

      if ((disableMsg != "") && (disableMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "BlockPlaceDisable", disableMsg);
      }

      event.getPlayer().updateInventory();
      event.setCancelled(true);
    }
  }

  @EventHandler
  public void onFall(EntityDamageEvent event) {
    Entity entity = event.getEntity();
    if ((entity instanceof Player)) {
      Player player = (Player)entity;
      if ((event.getCause() == EntityDamageEvent.DamageCause.FALL) && 
        (this.fallingPlayers.contains(player.getName()))) {
        int damage = (int) event.getDamage();
        int health =  (int) player.getHealth();
        int minhealth = Settings.minimumHealthFromFall();

        if (health - damage < minhealth)
          player.setHealth(minhealth);
        else {
          player.damage(damage);
        }
        this.fallingPlayers.remove(player.getName());
        if (minhealth > 0)
          event.setCancelled(true);
      }
    }
  }

  @EventHandler
  public void onPlayerBucketFill(PlayerBucketFillEvent event)
  {
    if ((event.getPlayer().getAllowFlight()) && (Settings.bucketFillBlocked()) && (!event.getPlayer().hasPermission("flydisable.exempt.bucketfill"))) {
      String warningMsg = Settings.getBucketFillWarningMsg();
      String disableMsg = Settings.getBucketFillDisableMsg();
      if ((warningMsg != "") && (warningMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "BucketFillWarning", warningMsg);
      }

      if ((disableMsg != "") && (disableMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "BucketFillDisable", disableMsg);
      }

      event.getPlayer().updateInventory();

      World world = event.getBlockClicked().getLocation().getWorld();
      Chunk chunk = world.getChunkAt(event.getBlockClicked().getLocation());
      world.refreshChunk(chunk.getX(), chunk.getZ());
      event.setCancelled(true);
    }
  }

  @EventHandler
  public void onPlayerBucketEmpty(PlayerBucketEmptyEvent event)
  {
    if ((event.getPlayer().getAllowFlight()) && (Settings.bucketEmptyBlocked()) && (!event.getPlayer().hasPermission("flydisable.exempt.bucketempty"))) {
      String warningMsg = Settings.getBucketEmptyWarningMsg();
      String disableMsg = Settings.getBucketEmptyDisableMsg();
      if ((warningMsg != "") && (warningMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "BucketEmptyWarning", warningMsg);
      }

      if ((disableMsg != "") && (disableMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "BucketEmptyDisable", disableMsg);
      }

      event.getPlayer().updateInventory();
      event.setCancelled(true);
    }
  }

  @EventHandler
  public void onPlayerPickupItem(PlayerPickupItemEvent event)
  {
    if ((event.getPlayer().getAllowFlight()) && (Settings.pickUpItemBlocked()) && (!event.getPlayer().hasPermission("flydisable.exempt.pickupitem"))) {
      String warningMsg = Settings.getPickUpItemWarningMsg();
      String disableMsg = Settings.getPickUpItemDisableMsg();
      if ((warningMsg != "") && (warningMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "PickUpItemWarning", warningMsg);
      }

      if ((disableMsg != "") && (disableMsg.length() > 0)) {
        MsgThread.getInstance().sendMsg(event.getPlayer(), "PickUpItemDisable", disableMsg);
      }

      event.getPlayer().updateInventory();
      event.setCancelled(true);
    }
  }

  @EventHandler
  public void onPlayerJoin(PlayerJoinEvent event)
  {
    if ((event.getPlayer().hasPermission("flydisable.admin")) && (Settings.allowedToNotifyAdmins()))
      event.getPlayer().sendMessage("&6FlyDisable Notification: New Version Is Available");
  }

