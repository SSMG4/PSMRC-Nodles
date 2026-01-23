# PSMRC-Nodles



<div align="center">
  <img src="public/img/psvita.gif" alt="PSMRC-Node" />
</div>



A dead Heroku website made to convert Minecraft Resource Packs to PSVita Resource Packs brought back to life with Render

Remember to support <a href="https://github.com/psiie/PSMRC-Node">the original project</a>, even though it might be dead now

All credits to <a href="https://github.com/psiie">Psiie/Indigoferra</a>, the creator of the original project. She also made <a href="https://github.com/psiie/PSMinecraft-Resource-Converter">PSMRC</a>, which is basically the same project but made as an application instead of a website, go check it out!

And lets not forget to link the website, which is available [here](https://psmrc-nodles.onrender.com)!

Now, onto the 

# Local Stuff

## Requirements
- Imagemagick installed on your command line
- node.js installed (v23.11.0 seems to work)

## How To Run
- `npm start`
- Navigate to localhost:3000
- Select the texture pack you want to convert
NOTE: due to copyright, there is no base texture pack in this repo. You will have to dump it yourself. You almost always want this. Without it, you will get all fallback blocks to be a basic simple color. Hold down shift when selecting these dumped textures. See the GIF animation included in this repo for more details.

# Instructions as seen on the home screen
Upload a modern resource pack to receive a PSVita Minecraft v1.84+ compatible texture pack.

Requires a modded PSVita with the rePatch plugin.

Extract zip in ux0:/rePatch

Optional: Not all texture conversions are implemented. To support fallback (legal reasons), upload the decrypted texture pack files terrain.png and items.png from the official game at the same time. (Hold Control to select multiple files). GIF example
(Possible IDs: PCSE00491, PCSB00560, PCSB00560, VCJS-10010)
(Folder Path: Common/res/TitleUpdate/res) 

# Example

![](public/img/how_to_select_multi_files.gif)

Latest Update: 2026.01.23
