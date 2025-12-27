# Hướng Dẫn Lập Trình Plugin Minecraft với ProtocolLib
## Phiên Bản Mới Nhất (v5.4.0)

---

## Mục Lục
1. [Giới Thiệu](#giới-thiệu)
2. [Yêu Cầu Hệ Thống](#yêu-cầu-hệ-thống)
3. [Cài Đặt & Thiết Lập](#cài-đặt--thiết-lập)
4. [Khái Niệm Cơ Bản](#khái-niệm-cơ-bản)
5. [API Chính](#api-chính)
6. [Ví Dụ Thực Tế](#ví-dụ-thực-tế)
7. [Xử Lý Gói Tin Nâng Cao](#xử-lý-gói-tin-nâng-cao)
8. [Best Practices](#best-practices)
9. [Khắc Phục Sự Cố](#khắc-phục-sự-cố)
10. [Tài Nguyên Bổ Sung](#tài-nguyên-bổ-sung)

---

## Giới Thiệu

**ProtocolLib** là thư viện mạnh mẽ cho phép các lập trình viên plugin Minecraft bắt (intercept), sửa đổi và gửi các gói tin mạng (packets) giữa server và client mà không cần truy cập vào code NMS (Net Minecraft Server) phức tạp.

### Lợi Ích Chính
- **Trừu Tượng Hóa NMS**: Không cần tương tác trực tiếp với code NMS dễ vỡ
- **Tương Thích Phiên Bản**: Tự động xử lý sự khác biệt giữa các version Minecraft
- **API Thân Thiện**: Giao diện đơn giản để đọc/ghi dữ liệu gói tin
- **Hiệu Năng Cao**: Được tối ưu hóa cho các server lớn
- **Hỗ Trợ Cộng Đồng**: Cập nhật nhanh cho các version Minecraft mới

### Các Trường Hợp Sử Dụng Phổ Biến
- Lọc/chỉnh sửa chat message
- Tạo fake entities hoặc blocks
- Anti-cheat detection
- Custom particle effects
- Giao tiếp giữa các server thông qua plugin channels
- Kiểm soát hành động player (di chuyển, tấn công, v.v.)

---

## Yêu Cầu Hệ Thống

### Phiên Bản Java
- **ProtocolLib v5.4.0**: Java 17 trở lên (bắt buộc)
- **ProtocolLib v5.3.0**: Java 8 trở lên
- **ProtocolLib v5.x**: Java 8+

### Server Hỗ Trợ
- Bukkit, Spigot, Paper
- Các fork của Bukkit API khác
- Minecraft 1.8 - 1.21.8

### IDE Khuyến Nghị
- IntelliJ IDEA
- Eclipse
- Visual Studio Code + Extension

---

## Cài Đặt & Thiết Lập

### 1. Thêm Dependency vào Maven

```xml
<dependency>
    <groupId>net.dmulloy2</groupId>
    <artifactId>protocollib</artifactId>
    <version>5.4.0</version>
</dependency>
```

### 2. Thêm Dependency vào Gradle

```gradle
dependencies {
    implementation 'net.dmulloy2:protocollib:5.4.0'
}
```

### 3. Cấu Hình plugin.yml

```yaml
name: MyProtocolPlugin
version: 1.0.0
main: com.example.MyPlugin
description: A plugin using ProtocolLib

# Khai báo dependency
depend:
  - ProtocolLib

# Hoặc soft-dependency (nếu có thể hoạt động mà không có ProtocolLib)
softdepend:
  - ProtocolLib
```

### 4. Tải ProtocolLib Plugin
- **CurseForge**: https://www.curseforge.com/minecraft/bukkit-plugins/protocollib
- **Hangar (PaperMC)**: https://hangar.papermc.io/dmulloy2/ProtocolLib
- **GitHub Releases**: https://github.com/dmulloy2/ProtocolLib/releases

---

## Khái Niệm Cơ Bản

### Packet (Gói Tin)
Đơn vị dữ liệu nhỏ nhất được truyền giữa client và server. Ví dụ:
- Chat message: `PacketType.Play.Client.CHAT`
- Spawn entity: `PacketType.Play.Server.SPAWN_ENTITY`
- Player movement: `PacketType.Play.Client.POSITION`

### Packet Event (Sự Kiện Gói Tin)
Được kích hoạt khi một gói tin được gửi/nhận. Cho phép plugin đọc, sửa đổi hoặc hủy gói tin.

### Listener Priority (Mức Độ Ưu Tiên)
```java
ListenerPriority.LOWEST      // Thực thi đầu tiên
ListenerPriority.LOW
ListenerPriority.NORMAL       // Mặc định
ListenerPriority.HIGH
ListenerPriority.HIGHEST     // Thực thi cuối cùng
```

### PacketContainer
Wrapper (bao bọc) cho packet object, cung cấp giao diện đồng nhất để:
- Đọc dữ liệu: `.getStrings().read(0)`
- Ghi dữ liệu: `.getStrings().write(0, "new value")`
- Clone packet: `.deepClone()`

---

## API Chính

### ProtocolManager - Entry Point

```java
// Lấy instance
private ProtocolManager protocolManager;

@Override
public void onEnable() {
    protocolManager = ProtocolLibrary.getProtocolManager();
}
```

### Các Phương Thức Chính

#### Đăng Ký Listener
```java
protocolManager.addPacketListener(new PacketAdapter(...) {
    @Override
    public void onPacketSending(PacketEvent event) { }
    
    @Override
    public void onPacketReceiving(PacketEvent event) { }
});
```

#### Gửi Packet Tùy Chỉnh
```java
try {
    protocolManager.sendServerPacket(player, packet);
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```

#### Broadcast Packet
```java
// Gửi đến tất cả players
protocolManager.broadcastServerPacket(packet);

// Gửi đến players theo khoảng cách
protocolManager.broadcastServerPacket(packet, location, distance);

// Gửi đến danh sách players
protocolManager.broadcastServerPacket(packet, playerList);
```

#### Tạo Packet Mới
```java
PacketContainer packet = protocolManager.createPacket(PacketType.Play.Server.EXPLOSION);
```

#### Xóa Listener
```java
protocolManager.removePacketListener(listener);
```

---

## Ví Dụ Thực Tế

### Ví Dụ 1: Lọc Chat Messages (Censorship)

```java
package com.example.plugins;

import org.bukkit.plugin.java.JavaPlugin;
import com.comphenix.protocol.ProtocolLibrary;
import com.comphenix.protocol.ProtocolManager;
import com.comphenix.protocol.events.PacketAdapter;
import com.comphenix.protocol.events.PacketEvent;
import com.comphenix.protocol.events.ListenerPriority;
import com.comphenix.protocol.packets.PacketType;

public class ChatCensorPlugin extends JavaPlugin {
    
    private ProtocolManager protocolManager;
    
    @Override
    public void onEnable() {
        protocolManager = ProtocolLibrary.getProtocolManager();
        setupChatListener();
        getLogger().info("Chat Censor Plugin enabled!");
    }
    
    private void setupChatListener() {
        protocolManager.addPacketListener(
            new PacketAdapter(this, ListenerPriority.NORMAL, PacketType.Play.Client.CHAT) {
                
                @Override
                public void onPacketReceiving(PacketEvent event) {
                    if (event.getPacketType() != PacketType.Play.Client.CHAT) {
                        return;
                    }
                    
                    PacketContainer packet = event.getPacket();
                    String message = packet.getStrings().read(0);
                    
                    // Lọc từ cấm
                    String filteredMessage = filterBadWords(message);
                    
                    if (!message.equals(filteredMessage)) {
                        packet.getStrings().write(0, filteredMessage);
                        event.getPlayer().sendMessage("§cYour message was filtered!");
                    }
                }
            }
        );
    }
    
    private String filterBadWords(String message) {
        String[] badWords = {"damn", "hell", "crap"};
        for (String word : badWords) {
            message = message.replaceAll("(?i)" + word, "***");
        }
        return message;
    }
}
```

### Ví Dụ 2: Fake Explosion

```java
private void sendFakeExplosion(Player player, double x, double y, double z, float radius) {
    PacketContainer explosion = protocolManager.createPacket(PacketType.Play.Server.EXPLOSION);
    
    // Thiết lập tọa độ
    explosion.getDoubles()
        .write(0, x)
        .write(1, y)
        .write(2, z);
    
    // Thiết lập bán kính
    explosion.getFloat().write(0, radius);
    
    try {
        protocolManager.sendServerPacket(player, explosion);
    } catch (InvocationTargetException e) {
        getLogger().warning("Cannot send explosion packet: " + e.getMessage());
    }
}

// Sử dụng:
// sendFakeExplosion(player, 100, 64, -200, 5.0f);
```

### Ví Dụ 3: Monitor Player Movement

```java
private void setupMovementListener() {
    protocolManager.addPacketListener(
        new PacketAdapter(this, ListenerPriority.NORMAL, 
                PacketType.Play.Client.POSITION,
                PacketType.Play.Client.POSITION_ROTATION) {
            
            @Override
            public void onPacketReceiving(PacketEvent event) {
                PacketContainer packet = event.getPacket();
                double x = packet.getDoubles().read(0);
                double y = packet.getDoubles().read(1);
                double z = packet.getDoubles().read(2);
                
                getLogger().info(event.getPlayer().getName() + " moved to: " + x + ", " + y + ", " + z);
                
                // Chặn player di chuyển vào vùng cấm
                if (isBlockedZone(x, z)) {
                    event.setCancelled(true);
                    event.getPlayer().teleport(event.getPlayer().getLocation());
                    event.getPlayer().sendMessage("§cYou cannot enter this zone!");
                }
            }
        }
    );
}

private boolean isBlockedZone(double x, double z) {
    // Ví dụ: vùng từ -100 đến 100
    return x > -100 && x < 100 && z > -100 && z < 100;
}
```

### Ví Dụ 4: Block Entity Metadata (Ẩn Armor)

```java
private void setupEntityMetadataListener() {
    protocolManager.addPacketListener(
        new PacketAdapter(this, ListenerPriority.NORMAL, 
                PacketType.Play.Server.ENTITY_METADATA) {
            
            @Override
            public void onPacketSending(PacketEvent event) {
                // Chỉ lọc armor equipment packets
                // Bỏ qua nếu không phải entity metadata
                getLogger().info("Entity metadata changed for entity: " + 
                    event.getPacket().getIntegers().read(0));
            }
        }
    );
}
```

---

## Xử Lý Gói Tin Nâng Cao

### Async Packet Processing (Xử Lý Không Đồng Bộ)

Đối với các tác vụ nặng, tránh block netty IO thread:

```java
private void setupAsyncListener() {
    protocolManager.addPacketListener(
        new PacketAdapter(this, ListenerPriority.NORMAL, PacketType.Play.Client.CHAT) {
            
            @Override
            public void onPacketReceiving(PacketEvent event) {
                PacketContainer packet = event.getPacket();
                Player player = event.getPlayer();
                
                // Đặt async context
                event.setAsynchronous(true);
                
                // Thực thi tác vụ nặng
                String message = packet.getStrings().read(0);
                checkMessageAsync(player, message);
            }
        }
    );
}

private void checkMessageAsync(Player player, String message) {
    // Tác vụ async (kiểm tra database, API call, v.v.)
    Bukkit.getScheduler().runTaskAsynchronously(this, () -> {
        boolean spam = checkIfSpam(message);
        if (spam) {
            Bukkit.getScheduler().runTask(this, () -> {
                player.sendMessage("§cYour message was flagged as spam!");
            });
        }
    });
}

private boolean checkIfSpam(String message) {
    // Logic kiểm tra spam
    return message.length() > 1000;
}
```

### Packet Cloning (Nhân Bản Gói Tin)

Khi broadcast packet đến nhiều players, cần clone để tránh shared instance issue:

```java
@Override
public void onPacketSending(PacketEvent event) {
    PacketContainer packet = event.getPacket().deepClone();
    
    // Giờ có thể sửa đổi packet cho mỗi player
    // mà không ảnh hưởng đến players khác
    packet.getStrings().write(0, "Modified for " + event.getPlayer().getName());
}
```

### Đọc/Ghi Dữ Liệu Phức Tạp

```java
// Đọc strings
String text = packet.getStrings().read(0);

// Đọc integers
int entityId = packet.getIntegers().read(0);

// Đọc doubles
double x = packet.getDoubles().read(0);

// Đọc boolean
boolean isOnGround = packet.getBooleans().read(0);

// Đọc NBT Data
NBTCompound nbt = packet.getNBTCompound().read(0);

// Đọc Item Stacks
ItemStack item = packet.getItemModifier().read(0);

// Chaining writes
packet.getStrings()
    .write(0, "New message")
    .write(1, "Another message");
```

### Custom Packet Types

```java
// Tạo packet hoàn toàn tùy chỉnh
PacketContainer customPacket = protocolManager.createPacket(PacketType.Play.Server.CUSTOM_PAYLOAD);
customPacket.getStrings().write(0, "channel:name");
customPacket.getByteArrays().write(0, customData);

protocolManager.sendServerPacket(player, customPacket);
```

---

## Best Practices

### 1. Kiểm Tra Null & Validate Dữ Liệu
```java
@Override
public void onPacketReceiving(PacketEvent event) {
    if (event == null || event.getPacket() == null) {
        return;
    }
    
    PacketContainer packet = event.getPacket();
    try {
        String message = packet.getStrings().read(0);
        if (message == null || message.isEmpty()) {
            return;
        }
        // Tiếp tục xử lý
    } catch (FieldAccessException e) {
        getLogger().warning("Error reading packet field: " + e.getMessage());
    }
}
```

### 2. Unregister Listeners Khi Disable Plugin
```java
@Override
public void onDisable() {
    protocolManager.removePacketListeners(this);
    getLogger().info("All packet listeners unregistered!");
}
```

### 3. Sử Dụng ListenerPriority Thích Hợp
```java
// Nếu cần chạy trước plugin khác
new PacketAdapter(this, ListenerPriority.HIGHEST, PacketType.Play.Client.CHAT) { }

// Nếu plugin khác cần chạy trước
new PacketAdapter(this, ListenerPriority.LOWEST, PacketType.Play.Client.CHAT) { }
```

### 4. Performance Optimization
```java
// ❌ KHÔNG: Lắng nghe tất cả packet
new PacketAdapter(this, ListenerPriority.NORMAL) { } // Sai!

// ✅ CÓ: Chỉ lắng nghe packet cần thiết
new PacketAdapter(this, ListenerPriority.NORMAL, 
    PacketType.Play.Client.CHAT,
    PacketType.Play.Client.POSITION) { }

// ✅ CÓ: Filter ngay trong listener
@Override
public void onPacketReceiving(PacketEvent event) {
    if (event.getPacketType() != PacketType.Play.Client.CHAT) {
        return; // Early exit
    }
    // Tiếp tục xử lý
}
```

### 5. Exception Handling
```java
@Override
public void onPacketSending(PacketEvent event) {
    try {
        PacketContainer packet = event.getPacket();
        // Xử lý packet
    } catch (FieldAccessException e) {
        getLogger().warning("Cannot access packet field: " + e.getMessage());
        e.printStackTrace();
    } catch (InvocationTargetException e) {
        getLogger().warning("Error sending packet: " + e.getMessage());
    } catch (Exception e) {
        getLogger().severe("Unexpected error: " + e.getMessage());
        e.printStackTrace();
    }
}
```

### 6. Kiểm Tra Phiên Bản Minecraft
```java
import org.bukkit.Bukkit;

private boolean isMinecraft121Plus() {
    try {
        Class.forName("com.comphenix.protocol.wrappers.WrappedChatComponent");
        return true;
    } catch (ClassNotFoundException e) {
        return false;
    }
}

// Hoặc check server version
int version = getServerVersion();
if (version >= 121) {
    // Dùng API cho 1.21+
} else {
    // Dùng API cho version cũ hơn
}

private int getServerVersion() {
    String version = Bukkit.getVersion();
    // Parse và extract version number
    return 121; // Example
}
```

---

## Khắc Phục Sự Cố

### Lỗi: "Unable to inject incoming channel"
**Nguyên Nhân**: Xung đột với plugin khác hoặc version mismatch
```java
// Kiểm tra ProtocolLib có enable không
if (!ProtocolLibrary.getProtocolManager().isShuttingDown()) {
    // An toàn để đăng ký listener
}
```

### Lỗi: "Cannot read field: NullPointerException"
**Nguyên Nhân**: Cố gắng đọc field không tồn tại
```java
// Thêm null check
try {
    String text = packet.getStrings().read(0);
    if (text != null) {
        // Xử lý
    }
} catch (ArrayIndexOutOfBoundsException | FieldAccessException e) {
    getLogger().warning("Cannot access packet field");
}
```

### Lỗi: "Packet listener not executed"
**Nguyên Nhân**: Listener không được đăng ký đúng
```java
// Đảm bảo ProtocolManager được lấy
private ProtocolManager protocolManager;

@Override
public void onEnable() {
    protocolManager = ProtocolLibrary.getProtocolManager();
    if (protocolManager == null) {
        getLogger().severe("ProtocolLib not found!");
        setEnabled(false);
        return;
    }
    
    // Đăng ký listener
    protocolManager.addPacketListener(...);
}
```

### Performance Issue: High CPU/Memory Usage
```java
// ✅ Giảm số packet listener
// ✅ Tắt listener khi không cần
// ✅ Sử dụng async processing cho tác vụ nặng
// ✅ Cache results khi có thể
// ✅ Profile với Spark: /spark profiler

// Kiểm tra số listener
getLogger().info("Active listeners: " + 
    protocolManager.getPacketListeners().size());
```

---

## Tài Nguyên Bổ Sung

### Tài Liệu Chính Thức
- **GitHub**: https://github.com/dmulloy2/ProtocolLib
- **JavaDoc**: https://ci.dmulloy2.net/job/ProtocolLib/javadoc/
- **Hangar**: https://hangar.papermc.io/dmulloy2/ProtocolLib
- **Wiki.vg**: https://wiki.vg/ (Protocol documentation)

### Packet Types Phổ Biến
```java
// Chat
PacketType.Play.Client.CHAT              // Player gửi message
PacketType.Play.Server.CHAT              // Server gửi message

// Movement
PacketType.Play.Client.POSITION          // Player vị trí
PacketType.Play.Client.POSITION_ROTATION // Player vị trí + rotation

// Entity
PacketType.Play.Server.SPAWN_ENTITY      // Spawn entity
PacketType.Play.Server.ENTITY_METADATA   // Update entity metadata
PacketType.Play.Server.ENTITY_TELEPORT   // Teleport entity

// Block
PacketType.Play.Client.BLOCK_PLACE       // Player đặt block
PacketType.Play.Server.BLOCK_CHANGE      // Server thay đổi block

// Inventory
PacketType.Play.Server.OPEN_WINDOW       // Mở inventory
PacketType.Play.Client.WINDOW_CLICK      // Click inventory

// Sound
PacketType.Play.Server.NAMED_SOUND_EFFECT // Phát sound
PacketType.Play.Server.ENTITY_SOUND      // Sound từ entity
```

### Communities & Discussions
- **Discord**: SpigotMC Server Development community
- **Reddit**: r/admincraft, r/Bukkit
- **Forums**: SpigotMC.org, PaperMC Community

### Tools & IDE Extensions
- **Maven**: Auto-download dependencies
- **Gradle**: Build tool tương tự
- **Intellij IDEA**: Hỗ trợ tốt nhất cho Bukkit development
- **Spark**: Performance profiling tool

### Plugins Sử Dụng ProtocolLib
- **Citizens**: NPC library
- **LibsDisguises**: Player/entity disguise
- **ViaVersion**: Protocol version compatibility
- **ActionHealth**: Damage indicator
- **OraxenItems**: Custom items

---

## Kết Luận

ProtocolLib là công cụ không thể thiếu cho các nhà phát triển plugin Minecraft muốn:
- Tương tác sâu với protocol Minecraft
- Tạo tính năng advanced mà Bukkit API không hỗ trợ
- Duy trì tương thích với nhiều version Minecraft
- Giảm thiểu complexity khi làm việc với NMS

**Lưu ý**: Luôn kiểm tra version tương thích, backup server trước khi update plugin, và test kỹ trước khi deploy vào production!

---

**Tài liệu này được tạo cho ProtocolLib v5.4.0**  
*Cập nhật lần cuối: Tháng 12 năm 2025*
