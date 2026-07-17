# PSMRC-Nodles



<div align="center">
  <img src="public/img/psvita.gif" alt="PSMRC-Node" />
</div>



A Minecraft Resource Pack to PSVita Resource Pack converter, originally a dead Heroku project, later revived on Render, now self-hosted.

Remember to support <a href="https://github.com/psiie/PSMRC-Node">the original project</a>, even though it might be dead now

All credits to <a href="https://github.com/psiie">Psiie/Indigoferra</a>, the creator of the original project. She also made <a href="https://github.com/psiie/PSMinecraft-Resource-Converter">PSMRC</a>, which is basically the same project but made as an application instead of a website, go check it out!

The website is available at [psmrc.ssmg4.dpdns.org](https://psmrc.ssmg4.dpdns.org)!

Now, onto the 

# Local Stuff

## Requirements
- ImageMagick installed on your command line (`convert`/`identify` binaries)
- Node.js installed (v24 LTS recommended — see `.nvmrc`)

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

# Deployment

See [DEPLOYMENT.md](DEPLOYMENT.md) for how this is deployed and kept running on the server. BE CAREFUL USING IT, YOU MIGHT GET CONFUSED.

Latest Update: 2026.07.16
