## Установка

```shell
cd
```

```shell
git clone https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging.git
```

```shell
ln -s ~/Klipper-Adaptive-Meshing-Purging/Configuration printer_data/config/KAMP
```

```shell
cp ~/Klipper-Adaptive-Meshing-Purging/Configuration/KAMP_Settings.cfg ~/printer_data/config/KAMP_Settings.cfg
```

## Настраиваем проверку обнавлений через интерфейс `Fluidd/Mainsail`

``` title="moonraker.conf"
[update_manager Klipper-Adaptive-Meshing-Purging]
type: git_repo
channel: dev
path: ~/Klipper-Adaptive-Meshing-Purging
origin: https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging.git
managed_services: klipper
primary_branch: main
```