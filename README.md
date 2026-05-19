The script 'test_resolvers' is an adaptation of the original work by Kris Lowet  
(https://gist.github.com/KrisLowet/675ba34e682c6d2afbc53fc317b41e85)  
for the development of my undergraduate thesis project.

The main structure was preserved; however, several modifications were implemented to support reproducible academic experimentation.

## Main Modifications

- Updated the evaluated resolvers, including only the following resolvers:
  - Cloudflare (`1.1.1.2`)
  - CleanBrowsing
  - AdGuard
  - Quad9

- Expanded the sinkhole IP set by adding AdGuard sinkhole IPs:
  - `94.140.14.33`
  - `94.140.14.35`
  - `94.140.14.15`

  This prevents blocking redirections from being interpreted as valid DNS resolutions.

- Processed each source independently before merging, ensuring greater consistency during data cleaning.

- Removed duplicates while preserving the original order using:

```bash
awk '!seen[$1]++'
