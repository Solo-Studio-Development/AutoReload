# AutoReload

This is a repository that makes life easier for you and your plugin users.

## 1️⃣ ConfigManager.class

Use this class to create a basic configuration manager class.

```java
package net.solostudio.itemforge.managers;

import lombok.Getter;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import net.solostudio.itemforge.processor.MessageProcessor;
import net.solostudio.itemforge.utils.LoggerUtils;
import org.bukkit.configuration.ConfigurationSection;
import org.bukkit.configuration.file.YamlConfiguration;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;

import java.io.File;
import java.io.IOException;
import java.util.List;

@Slf4j
@RequiredArgsConstructor
public class ConfigurationManager {

    @Getter private YamlConfiguration yml;
    @Getter private String name;
    private File config;

    public ConfigurationManager(@NotNull String dir, @NotNull String name) {
        File file = new File(dir);

        if (!file.exists()) {
            if (!file.mkdirs()) return;
        }

        config = new File(dir, name + ".yml");

        if (!config.exists()) {
            try {
                if (!config.createNewFile()) return;
            } catch (IOException exception) {
                LoggerUtils.error(exception.getMessage());
            }
        }

        yml = YamlConfiguration.loadConfiguration(config);

        yml.options().copyDefaults(true);

        this.name = name;
    }

    public void reload() {
        yml = YamlConfiguration.loadConfiguration(config);

        save();
    }

    public void set(@NotNull String path, @Nullable Object value) {
        yml.set(path, value);
        save();
    }

    public void save() {
        try {
            yml.save(config);
        } catch (IOException exception) {
            LoggerUtils.error(exception.getMessage());
        }
    }

    public List<String> getList(@NotNull String path) {
        return yml.getStringList(path)
                .stream()
                //.map(MessageProcessor::process)
                .toList();

    }

    public boolean getBoolean(@NotNull String path) {
        return yml.getBoolean(path);
    }

    public int getInt(@NotNull String path) {
        return yml.getInt(path);
    }

    public String getString(@NotNull String path) {
        return yml.getString(path);
    }

    public @Nullable ConfigurationSection getSection(@NotNull String path) {
        return yml.getConfigurationSection(path);
    }

    public void setName(@NotNull String name) {
        this.name = name;
    }
}

```

## 2️⃣ Create a configuration file AND registering the auto reload system

```java
package net.solostudio.itemforge.data;

import net.solostudio.itemforge.ItemForge;
import net.solostudio.itemforge.enums.keys.MessageKeys;
import net.solostudio.itemforge.managers.ConfigurationManager;
import net.solostudio.itemforge.utils.LoggerUtils;

import java.io.File;
import java.nio.file.*;

public class DataFile extends ConfigurationManager {
    private WatchService watchService;
    private long lastModificationTime;

    public DataFile() {
        super(ItemForge.getInstance().getDataFolder().getPath() + File.separator + "data", "items");
        save();
        startFileWatcher();
    }

    private void startFileWatcher() {
        try {
            File dataFile = new File(ItemForge.getInstance().getDataFolder().getPath() + File.separator + "data" + File.separator + "items.yml");
            this.lastModificationTime = dataFile.lastModified();

            Path path = Paths.get(ItemForge.getInstance().getDataFolder().getAbsolutePath(), "data");
            watchService = FileSystems.getDefault().newWatchService();

            path.register(watchService,
                    StandardWatchEventKinds.ENTRY_MODIFY,
                    StandardWatchEventKinds.ENTRY_CREATE,
                    StandardWatchEventKinds.ENTRY_DELETE);

            ItemForge.getInstance().getScheduler().runTaskAsynchronously(() -> {
                WatchKey key;

                try {
                    key = watchService.take();
                } catch (InterruptedException exception) { throw new RuntimeException(exception); }

                key.pollEvents().forEach(event -> {
                    Path changedFile = (Path) event.context();

                    if (changedFile.endsWith("items.yml")) {
                        File currentFile = new File(ItemForge.getInstance().getDataFolder().getPath() + File.separator + "data" + File.separator + "items.yml");
                        long currentModificationTime = currentFile.lastModified();

                        if (currentModificationTime != lastModificationTime) {
                            lastModificationTime = currentModificationTime;
                            reload();
                            LoggerUtils.info(MessageKeys.RELOAD.getMessage().replace("&", ""));
                        }
                    }

                    key.reset();
                });
            });
        } catch (Exception exception) {
            LoggerUtils.error(exception.getMessage());
        }
    }

    public void shutdownWatcher() {
        try {
            if (watchService != null) watchService.close();
        } catch (Exception exception) {
            LoggerUtils.error(exception.getMessage());
        }
    }
}

```

## 3️⃣ Paste this in your main class
### 1️⃣ Registering the file
```java
@Getter private DataFile dataFile;
```

### 2️⃣ Initialization.

```java
@Override
    public void onEnable() {
        saveResource("data/items.yml", true);
        dataFile = new DataFile();
    }
```

### 3️⃣ Make it to stop.

```java
@Override
    public void onDisable() {
        dataFile.shutdownWatcher();
    }
```

## 🤔 Discord
### You can find different infos here. I'd appreciate it you'd join thanks:)
[Click here to join](https://discord.gg/CxawDwDZtd)

## ⚡ Github
### You can find different repos here. I'd appreciate it you'd follow me here thanks:)
[Click here to follow](https://github.com/User-19fff)
