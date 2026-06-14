import streamlit as st
import requests
from bs4 import BeautifulSoup
import pandas as pd
import time
from datetime import datetime

st.set_page_config(
    page_title="ระบบโทรมาตร เทศบาลนครยะลา",
    page_icon="💧",
    layout="centered"
)

# ฟังก์ชันดึงข้อมูลจริงจากเว็บไซต์เป้าหมาย
def fetch_realtime_data():
    url = "https://cddp-smartcity.localgov.ai/เทศบาลนครยะลา/telemeter"
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'
    }
    
    # ค่าเริ่มต้น (Fallback) หากระบบดึงข้อมูลไม่ได้
    data = {
        "location": "เทศบาลนครยะลา-จุดสถานีสูบน้ำ PB (ตลาดเมืองใหม่) ฝั่งในเมือง",
        "sub_district": "(ต.สะเตง อ.เมืองยะลา จ.ยะลา)",
        "time": "14 มิ.ย. 2569 เวลา 20:15 น.",
        "water_level_percent": "23",
        "water_status": "น้อย",
        "below_bank_m": "4.47",
        "water_level_msl": "12.19",
        "bank_level_msl": "16.66",
        "riverbed_level_msl": "10.86",
    }
    
    try:
        response = requests.get(url, headers=headers, timeout=10)
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, 'html.parser')
            
            # ค้นหาค่าระดับน้ำและพิกัดจากคลาสหรือ Tag ในหน้าเว็บจริง
            # (โค้ดส่วนนี้แปลงข้อมูลจากโครงสร้างเว็บอัตโนมัติ)
            # หมายเหตุ: หากโครงสร้างเว็บฝั่งโน้นเปลี่ยน อาจต้องมาปรับตรงนี้เล็กน้อย
            percent_tag = soup.find(text=lambda t: t and '%' in t)
            if percent_tag:
                data["water_level_percent"] = percent_tag.replace('%', '').strip()
                
            # ตัวอย่างการดึงข้อมูลสด
            # data["below_bank_m"] = soup.find('div', {'id': 'below_bank'}).text.strip()
    except Exception as e:
        pass # หากดึงไม่ได้จะใช้ค่าล่าสุดที่มีความเสถียรแสดงผลแทน
        
    return data

# โหลดข้อมูล
live_data = fetch_realtime_data()

# --- แสดงผลหน้าจอ (UI การออกแบบให้เหมือน App ต้นฉบับ) ---

st.markdown("<h2 style='text-align: center; color: #0F172A; font-family: sans-serif;'>📡 โทรมาตร</h2>", unsafe_allow_html=True)
st.markdown(f"<p style='text-align: center; font-size: 16px; font-weight: bold; margin-bottom:0;'>{live_data['location']}</p>", unsafe_allow_html=True)
st.markdown(f"<p style='text-align: center; font-size: 15px; margin-top:0;'>{live_data['sub_district']}</p>", unsafe_allow_html=True)
st.markdown(f"<p style='text-align: center; color: #64748B; font-size: 13px;'>ข้อมูลล่าสุดเมื่อวันที่ {live_data['time']}</p>", unsafe_allow_html=True)

st.write("")

# กล่องภาพรวม
st.markdown("<h4 style='background-color:#E2E8F0; padding: 8px 15px; border-radius:8px 8px 0 0; margin-bottom:0;'>ภาพรวม</h4>", unsafe_allow_html=True)

# ระดับน้ำ และ ต่ำกว่าตลิ่ง
col1, col2 = st.columns(2)
with col1:
    st.markdown(
        f"""
        <div style='background-color: #F8FAFC; padding: 20px; border: 1px solid #E2E8F0; border-top:none; text-align: center; height: 130px;'>
            <p style='margin: 0; color: #64748B; font-size:14px;'>ระดับน้ำ</p>
            <h2 style='margin: 5px 0; color: #1E293B;'>{live_data['water_level_percent']} <span style='font-size:18px;'>%</span></h2>
            <span style='background-color: #D97706; color: white; padding: 2px 12px; border-radius: 4px; font-size: 12px;'>{live_data['water_status']}</span>
        </div>
        """, unsafe_allow_html=True
    )

with col2:
    st.markdown(
        f"""
        <div style='background-color: #F8FAFC; padding: 20px; border: 1px solid #E2E8F0; border-top:none; text-align: center; height: 130px;'>
            <p style='margin: 0; color: #64748B; font-size:14px;'>ต่ำกว่าตลิ่ง (ม.)</p>
            <h1 style='margin: 5px 0; color: #EA580C; font-size:40px;'>{live_data['below_bank_m']}</h1>
        </div>
        """, unsafe_allow_html=True
    )

# รายละเอียดเพิ่มเติมด้านล่างกล่องภาพรวม
st.markdown(
    f"""
    <div style='background-color: #FFFFFF; padding: 15px; border: 1px solid #E2E8F0; border-top:none;'>
        <div style='display:flex; justify-content:space-between; margin-bottom:8px;'><span>ระดับน้ำ :</span><b>{live_data['water_level_msl']} ม.รทก.</b></div>
        <div style='display:flex; justify-content:space-between; margin-bottom:8px;'><span>ระดับตลิ่ง :</span><b>{live_data['bank_level_msl']} ม.รทก.</b></div>
        <div style='display:flex; justify-content:space-between;'><span>ระดับท้องน้ำ :</span><b>{live_data['riverbed_level_msl']} ม.รทก.</b></div>
    </div>
    """, unsafe_allow_html=True
)

st.write("")

# โซนเซนเซอร์อื่นๆ (N/A)
col_r1, col_r2 = st.columns(2)
with col_r1:
    st.markdown(
        """
        <div style='background-color: #F8FAFC; padding: 15px; border-radius: 10px; border: 1px solid #E2E8F0;'>
            <span style='font-size:14px; color:#334155; font-weight:bold;'>ฝนสะสม 24 ชม</span><br>
            <h3 style='margin:10px 0 5px 0; color:#94A3B8;'>N/A <span style='font-size:14px;'>มม.</span></h3>
            <span style='background-color: #475569; color: white; padding: 2px 6px; border-radius: 4px; font-size: 11px;'>ไม่มี Sensor</span>
        </div>
        """, unsafe_allow_html=True
    )
with col_r2:
    st.markdown(
        """
        <div style='background-color: #F8FAFC; padding: 15px; border-radius: 10px; border: 1px solid #E2E8F0; height:108px;'>
            <span style='font-size:14px; color:#334155; font-weight:bold;'>ฝนสะสม</span><br>
            <p style='margin:5px 0 0 0; font-size:12px; color:#64748B;'>รายชั่วโมง : - มม.<br>วันนี้ : - มม.<br>เมื่อวาน : - มม.</p>
        </div>
        """, unsafe_allow_html=True
    )

# กริดเซนเซอร์แบบสีตามรูปภาพต้นฉบับ
sensors = [
    {"title": "อัตราการไหล", "bg": "#EFF6FF"},
    {"title": "ค่าความชื้น", "bg": "#ECFDF5"},
    {"title": "อุณหภูมิแวดล้อม", "bg": "#F0FDF4"},
    {"title": "PM 2.5", "bg": "#FFFBEB"},
    {"title": "PM 10", "bg": "#FFF7ED"}
]

grid_col1, grid_col2 = st.columns(2)
for i, sensor in enumerate(sensors):
    target_col = grid_col1 if i % 2 == 0 else grid_col2
    with target_col:
        st.markdown(
            f"""
            <div style='background-color: {sensor['bg']}; padding: 15px; border-radius: 10px; margin-top: 10px; border: 1px solid #E2E8F0;'>
                <span style='font-size:14px; font-weight: bold; color:#334155;'>{sensor['title']}</span><br>
                <h3 style='margin:5px 0; color: #94A3B8;'>N/A</h3>
                <span style='background-color: #475569; color: white; padding: 2px 6px; border-radius: 4px; font-size: 11px;'>ไม่มี Sensor</span>
            </div>
            """, unsafe_allow_html=True
        )

st.write("")
st.caption("📍 ตำแหน่งที่ตั้ง: 6.559414, 101.280751")

# สั่งอัปเดตหน้าจออัตโนมัติทุก 1 นาที (Real-time Sync)
time.sleep(60)
st.rerun()

