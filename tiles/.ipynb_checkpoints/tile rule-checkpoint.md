NextSeq Sytstem

LaneCount = 4
SurfaceCount = 2
SwathCount = 1
SectionPerLane = 3
LanePerSection = 2
TileCount = 12

```
TILES = [(L+1, f"{Su+1}{Sw+1}{L//LanePerSection*SectionPerLane+SPL+1}{T+1:02d}")
         for L in range(LaneCount)
         for Su in range(SurfaceCount)
         for Sw in range(SwathCount)
         for SPL in range(SectionPerLane)
         for T in range(TileCount)]
```