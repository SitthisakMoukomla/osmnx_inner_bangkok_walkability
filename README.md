# 🚶 Bangkok Walkability H3 Hexbin Map

> วิเคราะห์ความเป็นมิตรต่อคนเดินเท้า (Walkability) ของกรุงเทพฯ เขตชั้นใน ด้วย H3 Hexagonal Grid + ข้อมูลจริงจาก OpenStreetMap

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![OSMnx](https://img.shields.io/badge/OSMnx-2.1.0-green)
![H3](https://img.shields.io/badge/H3-Resolution_9-orange)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

## TL;DR

ดึง POI และ walking network จาก OSM ผ่าน OSMnx 2.1.0 → แบ่งพื้นที่เป็น H3 hexagon (res 9, ~174m) → คำนวณ walkability score จาก 6 ตัวแปร → render เป็น interactive 3D map ด้วย PyDeck

## พื้นที่ศึกษา

กรุงเทพฯ เขตชั้นใน 15 เขต:

พระนคร · ดุสิต · ป้อมปราบศัตรูพ่าย · สัมพันธวงศ์ · บางรัก · สาทร · บางคอแหลม · ปทุมวัน · ราชเทวี · พญาไท · ดินแดง · ห้วยขวาง · วัฒนา · คลองเตย · จตุจักร

## ตัวแปร Walkability (6 ตัว)

| ตัวแปร | OSM Tag | Weight | หมายเหตุ |
|---|---|---|---|
| ทางเท้า/ทางเดิน | `highway=footway\|pedestrian\|path\|steps` | 0.25 | ความยาวรวม (m) ต่อ hex |
| ร้านอาหาร | `amenity=restaurant` | 0.15 | จำนวนต่อ hex |
| คาเฟ่ | `amenity=cafe` | 0.15 | จำนวนต่อ hex |
| ร้านสะดวกซื้อ | `shop=convenience` | 0.15 | จำนวนต่อ hex |
| สวนสาธารณะ | `leisure=park` | 0.15 | จำนวนต่อ hex |
| ป้ายรถเมล์/สถานี | `highway=bus_stop`, `railway=station\|halt` | 0.15 | จำนวนต่อ hex |

Score คำนวณจาก min-max normalization ของแต่ละตัวแปร แล้วรวม weighted sum → scale เป็น 0–100

## Tech Stack

- **[OSMnx 2.1.0](https://osmnx.readthedocs.io/)** — ดึงข้อมูลจาก OpenStreetMap Overpass API
- **[H3](https://h3geo.org/)** (Uber) — Hexagonal hierarchical spatial index, resolution 9
- **[PyDeck](https://deckgl.readthedocs.io/)** / **[deck.gl](https://deck.gl/)** — H3HexagonLayer visualization + CARTO basemap (ฟรี ไม่ต้อง API key)
- **[Folium](https://python-visualization.github.io/folium/)** — Alternative renderer (Leaflet + CartoDB dark_matter tile)
- **GeoPandas** + **Shapely** — Spatial operations
- **Google Colab** — Runtime environment

## Quick Start

### Google Colab (แนะนำ)

1. Upload `bkk_walkability_h3.ipynb` ไปที่ [Google Colab](https://colab.research.google.com/)
2. รัน Cell ทีละอัน (ทั้งหมด ~5-8 นาที เพราะต้องดึงข้อมูลจาก Overpass API)
3. ได้ interactive map + export `.html` และ `.geojson`

### Local

```bash
pip install osmnx h3 pydeck geopandas shapely
jupyter notebook bkk_walkability_h3.ipynb
```

## Output

| ไฟล์ | รายละเอียด |
|---|---|
| `bkk_walkability_h3.html` | Interactive 3D map (PyDeck + CARTO basemap) เปิดใน browser ได้เลย |
| `bkk_walkability_folium.html` | Alternative 2D map (Folium + Leaflet) กันพลาด |
| `bkk_walkability_h3.geojson` | H3 polygons + score เปิดใน QGIS / Kepler.gl |

## โครงสร้างโปรเจค

```
.
├── README.md
├── bkk_walkability_h3.ipynb    # Colab notebook หลัก
├── assets/
│   └── bkk_walkability_preview.png
└── output/
    ├── bkk_walkability_h3.html
    └── bkk_walkability_h3.geojson
```

## ปรับแต่ง

### เปลี่ยน Weight

แก้ใน Cell ที่ 6 ของ notebook:

```python
WEIGHTS = {
    'footway_m_norm':    0.25,  # ทางเท้า — สำคัญสุด
    'restaurant_norm':   0.15,
    'cafe_norm':         0.15,
    'convenience_norm':  0.15,
    'park_norm':         0.15,
    'transit_norm':      0.15,
}
```

### เปลี่ยน H3 Resolution

```python
H3_RES = 8   # ~460m — เร็วกว่า hex น้อยกว่า
H3_RES = 9   # ~174m — default ละเอียดดี
H3_RES = 10  # ~66m  — ละเอียดมาก ช้าขึ้น
```

### เพิ่มเขต

เพิ่มชื่อเขตใน list `INNER_DISTRICTS` ได้ ใช้ชื่อภาษาอังกฤษตาม OSM Nominatim:

```python
INNER_DISTRICTS.append("Bang Sue, Bangkok, Thailand")
```

## Known Issues

- **Overpass API timeout** — ถ้าดึงข้อมูลไม่ได้ ลองเพิ่ม `ox.settings.timeout = 600` หรือลดจำนวนเขต
- **OSM data coverage** — ข้อมูล OSM ในกรุงเทพฯ ไม่ครบ 100% โดยเฉพาะทางเท้า และร้านค้าบางส่วน ผลลัพธ์จึงอาจ underestimate walkability ในบางพื้นที่
- **h3 v3 vs v4** — Notebook มี try/except fallback รองรับทั้ง 2 version

## References

- Boeing, G. (2025). Modeling and Analyzing Urban Networks and Amenities with OSMnx. *Geographical Analysis*, 57(4), 567–577. doi:10.1111/gean.70009
- Uber H3: https://h3geo.org/
- OpenStreetMap: https://www.openstreetmap.org/copyright

## License

MIT — ใช้ได้อิสระ ดัดแปลงได้ ขอแค่ให้เครดิต

## Author

Sitthisak Moukomla (Sith)
Department of Geography, Faculty of Liberal Arts, Thammasat University
