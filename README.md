Plug.dj-bukkit
==============

Expermental alteration of plugin for plug.dj

package main.alexhaslam.frag;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.InetSocketAddress;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLConnection;
import java.util.logging.Logger;
import java.util.regex.Pattern;
import org.bukkit.Bukkit;
import org.bukkit.ChatColor;
import org.bukkit.Server;
import org.bukkit.command.Command;
import org.bukkit.command.CommandSender;
import org.bukkit.configuration.file.FileConfiguration;
import org.bukkit.configuration.file.FileConfigurationOptions;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.EventPriority;
import org.bukkit.event.Listener;
import org.bukkit.event.player.PlayerChatEvent;
import org.bukkit.plugin.PluginManager;
import org.bukkit.plugin.java.JavaPlugin;
import org.bukkit.scheduler.BukkitScheduler;
import sun.misc.BASE64Encoder;

public class FragRadio extends JavaPlugin
  implements Listener
{
  FileConfiguration config;
  public String logPrefix = "[FragRadio] ";
  public Logger log = Logger.getLogger("Minecraft");
  FragRadio plugin = this;
  public String dj = "";
  public String track = "";
  public String olddj = "";
  public String oldtrack = "";
  public String Sip;
  public int Sport;
  public String Sname;
  public boolean Obroad;

  public void loadConfiguration()
  {
    this.plugin.getConfig().addDefault("FragRadio.info.address", getServer().getIp());
    this.plugin.getConfig().addDefault("FragRadio.info.port", Integer.valueOf(getServer().getPort()));
    this.plugin.getConfig().addDefault("FragRadio.info.name", "Deafult name");
    this.plugin.getConfig().addDefault("FragRadio.options.broadcast", Boolean.valueOf(true));

    this.plugin.getConfig().options().copyDefaults(true);

    this.Sip = this.plugin.getConfig().getString("FragRadio.info.address");
    this.Sport = this.plugin.getConfig().getInt("FragRadio.info.port");
    this.Sname = this.plugin.getConfig().getString("FragRadio.info.name");
    this.Obroad = this.plugin.getConfig().getBoolean("FragRadio.options.broadcast");

    this.plugin.saveConfig();
  }

  public void onDisable()
  {
    this.log.info(this.logPrefix + " is disabled");
  }

  public void onEnable()
  {
    this.plugin.getServer().getPluginManager().registerEvents(this, this.plugin);

    loadConfiguration();
    this.log.info(this.logPrefix + " is enabled");
    this.plugin.getServer().getScheduler().scheduleAsyncRepeatingTask(this.plugin, new Runnable()
    {
      public void run() {
        FragRadio.this.erLogCall(FragRadio.this.Sip, FragRadio.this.Sport, FragRadio.this.Sname, FragRadio.this.getServer().getOnlinePlayers().length, FragRadio.this.getServer().getMaxPlayers());
      }
    }
    , 40L, 2400L);
    this.plugin.getServer().getScheduler().scheduleAsyncRepeatingTask(this.plugin, new Runnable()
    {
      public void run() {
        FragRadio.this.grab();
      }
    }
    , 40L, 400L);
  }

  public String encodeStringBase64(String s)
  {
    BASE64Encoder encoder = new BASE64Encoder();
    return encoder.encodeBuffer(s.getBytes()).trim();
  }

  public boolean erLogCall(String servIP, int servPort, String servName, int servOnline, int servSlot) {
    String encodedString2 = servIP;
    String encodedString3 = Integer.toString(servPort);
    String encodedString4 = servName;
    String encodedString5 = Integer.toString(servSlot);
    String encodedString6 = Integer.toString(servOnline);
    String erMT = "ip=" + encodedString2.replace("&", "%26").replace("*", "%2A").replace("{", "%7B").replace("}", "%7D").replace("\\", "%5C").replace(":", "%3A").replace("<", "%3C").replace(">", "%3E").replace("?", "%3F").replace("/", "%2F").replace("+", "%2B") + "&port=" + encodedString3.replace("#", "%23").replace("%", "%25").replace("&", "%26").replace("*", "%2A").replace("{", "%7B").replace("}", "%7D").replace("\\", "%5C").replace(":", "%3A").replace("<", "%3C").replace(">", "%3E").replace("?", "%3F").replace("/", "%2F").replace("+", "%2B") + "&name=" + encodedString4.replace("%", "%25").replace(" ", "%20").replace("#", "%23").replace("&", "%26").replace("*", "%2A").replace("{", "%7B").replace("}", "%7D").replace("\\", "%5C").replace(":", "%3A").replace("<", "%3C").replace(">", "%3E").replace("?", "%3F").replace("/", "%2F").replace("+", "%2B") + "&maxplayers=" + encodedString5.replace("#", "%23").replace("%", "%25").replace("&", "%26").replace("*", "%2A").replace("{", "%7B").replace("}", "%7D").replace("\\", "%5C").replace(":", "%3A").replace("<", "%3C").replace(">", "%3E").replace("?", "%3F").replace("/", "%2F").replace("+", "%2B") + "&players=" + encodedString6.replace("#", "%23").replace("%", "%25").replace("&", "%26").replace("*", "%2A").replace("{", "%7B").replace("}", "%7D").replace("\\", "%5C").replace(":", "%3A").replace("<", "%3C").replace(">", "%3E").replace("?", "%3F").replace("/", "%2F").replace("+", "%2B");
    this.log.info(this.logPrefix + erMT);
    URL call = null;
    URLConnection callConnect = null;
    InputStreamReader inStream = null;
    BufferedReader read = null;
    try
    {
      call = new URL("http://FragRadio.co.uk/mc/update?" + erMT);

      callConnect = call.openConnection();
      inStream = new InputStreamReader(callConnect.getInputStream());
      read = new BufferedReader(inStream);

      String outline = read.readLine();
      if (outline.trim().toString() != "fail")
      {
        if (getServer().getOnlinePlayers().length != 0) {
          this.log.info(this.logPrefix + " is logging");
        }
        return true;
      }

      return false;
    }
    catch (MalformedURLException e)
    {
      this.log.info(this.logPrefix + " failed to create url");
    } catch (IOException localIOException) {
    }
    return false;
  }

  public void cast(String list) {
    Bukkit.getServer().broadcastMessage(ChatColor.GOLD + "[FragRadio] " + ChatColor.WHITE + list.toString());
  }

  public void grab() {
    URL erp = null;
    URLConnection erpConnect = null;
    InputStreamReader inStream = null;
    BufferedReader buff = null;
    try
    {
      erp = new URL("http://noc.get-sourced.net/frag.php");
      erpConnect = erp.openConnection();
      inStream = new InputStreamReader(erpConnect.getInputStream());
      buff = new BufferedReader(inStream);

      String prefix = "Dj: \nSong: ";
      String[] pre = Pattern.compile("\n").split(prefix.trim().toString());
      int i = 0;
      while (true)
      {
        String nextLine = buff.readLine();
        if (nextLine == null) {
          break;
        }
        String inf = nextLine.trim().toString();
        if (i == 0) {
          this.dj = (pre[0] + inf.toString());
          if (!this.dj.equals(this.olddj))
          {
            if (this.Obroad) cast(this.dj);

            this.olddj = this.dj;
          }
        }
        else if (i == 1) {
          this.track = (pre[1] + inf.toString());
          if (!this.track.equals(this.oldtrack))
          {
            if (this.Obroad) cast(this.track);

            this.oldtrack = this.track;
          }
        }

        i++;
      }
    }
    catch (IOException e) {
      e.printStackTrace();
    }
  }

  public void send(String frT) {
    URL erp = null;
    URLConnection erpConnect = null;
    InputStreamReader inStream = null;

    BufferedReader buff = null;
    try
    {
      erp = new URL("http://www.fragradio.co.uk/ingame/request.php" + frT);
      erpConnect = erp.openConnection();
      inStream = new InputStreamReader(erpConnect.getInputStream());
      buff = new BufferedReader(inStream);
    }
    catch (IOException e) {
      e.printStackTrace();
    }
  }

  public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
    if (!(sender instanceof Player)) {
      return false;
    }

    if (label.equalsIgnoreCase("request")) {
      StringBuilder rt = new StringBuilder();
      for (String arg : args) {
        rt.append(arg + " ");
      }
      String encodedString2 = encodeStringBase64("shout");
      String encodedString3 = encodeStringBase64(rt.toString());
      String encodedString4 = encodeStringBase64(((Player)sender).getAddress().toString());
      String encodedString5 = encodeStringBase64(((Player)sender).getName());
      String encodedString6 = encodeStringBase64("nosteamid");
      String encodedString7 = encodeStringBase64(Bukkit.getServer().getIp() + ":" + Bukkit.getServer().getPort());
      String frT = "?type=" + encodedString2 + "&content=" + encodedString3 + "&playersip=" + encodedString4 + "&playersname=" + encodedString5 + "&playerssteam=" + encodedString6 + "&serversip=" + encodedString7;
      send(frT);
      sender.sendMessage(ChatColor.GOLD + "[FragRadio] " + "Request sent.");
    }
    if (label.equalsIgnoreCase("djftw")) {
      StringBuilder rt = new StringBuilder();
      for (String arg : args) {
        rt.append(arg + " ");
      }
      String encodedString2 = encodeStringBase64("dj");
      String encodedString3 = encodeStringBase64("ftw");
      String encodedString4 = encodeStringBase64(((Player)sender).getAddress().toString());
      String encodedString5 = encodeStringBase64(((Player)sender).getName());
      String encodedString6 = encodeStringBase64("nosteamid");
      String encodedString7 = encodeStringBase64(Bukkit.getServer().getIp() + ":" + Bukkit.getServer().getPort());
      String frT = "?type=" + encodedString2 + "&content=" + encodedString3 + "&playersip=" + encodedString4 + "&playersname=" + encodedString5 + "&playerssteam=" + encodedString6 + "&serversip=" + encodedString7;
      send(frT);
    }

    if (label.equalsIgnoreCase("djftl")) {
      StringBuilder rt = new StringBuilder();
      for (String arg : args) {
        rt.append(arg + " ");
      }
      String encodedString2 = encodeStringBase64("dj");
      String encodedString3 = encodeStringBase64("ftl");
      String encodedString4 = encodeStringBase64(((Player)sender).getAddress().toString());
      String encodedString5 = encodeStringBase64(((Player)sender).getName());
      String encodedString6 = encodeStringBase64("nosteamid");
      String encodedString7 = encodeStringBase64(Bukkit.getServer().getIp() + ":" + Bukkit.getServer().getPort());
      String frT = "?type=" + encodedString2 + "&content=" + encodedString3 + "&playersip=" + encodedString4 + "&playersname=" + encodedString5 + "&playerssteam=" + encodedString6 + "&serversip=" + encodedString7;
      send(frT);
    }

    if (label.equalsIgnoreCase("choon")) {
      StringBuilder rt = new StringBuilder();
      for (String arg : args) {
        rt.append(arg + " ");
      }
      String encodedString2 = encodeStringBase64("song");
      String encodedString3 = encodeStringBase64("ftw");
      String encodedString4 = encodeStringBase64(((Player)sender).getAddress().toString());
      String encodedString5 = encodeStringBase64(((Player)sender).getName());
      String encodedString6 = encodeStringBase64("nosteamid");
      String encodedString7 = encodeStringBase64(Bukkit.getServer().getIp() + ":" + Bukkit.getServer().getPort());
      String frT = "?type=" + encodedString2 + "&content=" + encodedString3 + "&playersip=" + encodedString4 + "&playersname=" + encodedString5 + "&playerssteam=" + encodedString6 + "&serversip=" + encodedString7;
      send(frT);
    }
    if (label.equalsIgnoreCase("poon")) {
      StringBuilder rt = new StringBuilder();
      for (String arg : args) {
        rt.append(arg + " ");
      }
      String encodedString2 = encodeStringBase64("song");
      String encodedString3 = encodeStringBase64("ftl");
      String encodedString4 = encodeStringBase64(((Player)sender).getAddress().toString());
      String encodedString5 = encodeStringBase64(((Player)sender).getName());
      String encodedString6 = encodeStringBase64("nosteamid");
      String encodedString7 = encodeStringBase64(Bukkit.getServer().getIp() + ":" + Bukkit.getServer().getPort());
      String frT = "?type=" + encodedString2 + "&content=" + encodedString3 + "&playersip=" + encodedString4 + "&playersname=" + encodedString5 + "&playerssteam=" + encodedString6 + "&serversip=" + encodedString7;
      send(frT);
    }

    if (label.equalsIgnoreCase("shoutout")) {
      StringBuilder rt = new StringBuilder();
      for (String arg : args) {
        rt.append(arg + " ");
      }
      String encodedString2 = encodeStringBase64("shout");
      String encodedString3 = encodeStringBase64(rt.toString());
      String encodedString4 = encodeStringBase64(((Player)sender).getAddress().toString());
      String encodedString5 = encodeStringBase64(((Player)sender).getName());
      String encodedString6 = encodeStringBase64("nosteamid");
      String encodedString7 = encodeStringBase64(Bukkit.getServer().getIp() + ":" + Bukkit.getServer().getPort());
      String frT = "?type=" + encodedString2 + "&content=" + encodedString3 + "&playersip=" + encodedString4 + "&playersname=" + encodedString5 + "&playerssteam=" + encodedString6 + "&serversip=" + encodedString7;
      send(frT);
    }
    return true;
  }

  @EventHandler(priority=EventPriority.NORMAL)
  public void onPlayerChat(PlayerChatEvent event) {
    if (event.getMessage().startsWith("!dj"))
      this.plugin.getServer().getScheduler().scheduleSyncDelayedTask(this.plugin, new Runnable()
      {
        public void run() {
          Bukkit.getServer().broadcastMessage(ChatColor.YELLOW + "[FragRadio] " + ChatColor.WHITE + FragRadio.this.dj);
        }
      }
      , 1L);
    else if (event.getMessage().startsWith("!song"))
      this.plugin.getServer().getScheduler().scheduleSyncDelayedTask(this.plugin, new Runnable() {
        public void run() {
          Bukkit.getServer().broadcastMessage(ChatColor.GRAY + "[FragRadio] " + ChatColor.WHITE + FragRadio.this.track);
        }
      }
      , 1L);
  }
}