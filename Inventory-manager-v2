import streamlit as st
import qrcode
from PIL import Image
import os, json
import pandas as pd
import cv2, numpy as np

# Setup
st.set_page_config(layout="wide")
DATA_FILE = "data.json"
LOC_FILE = "locations.json"
os.makedirs("Items", exist_ok=True)

# Load or init
if os.path.exists(DATA_FILE):
    items = pd.read_json(DATA_FILE, orient="records")
else:
    items = pd.DataFrame(columns=["name","location","price","misc","barcode_img","picture_path"])
if os.path.exists(LOC_FILE):
    locations = json.load(open(LOC_FILE))
else:
    locations = []
st.session_state.setdefault("locations", locations)

# Save utilities
def save_items(): items.to_json(DATA_FILE, orient="records")
def save_locs(): json.dump(st.session_state.locations, open(LOC_FILE,"w"))

# Sidebar: Manage locations
st.sidebar.header("📍 Locations")
new_loc = st.sidebar.text_input("Add new location")
if st.sidebar.button("➕ Add"):
    if new_loc and new_loc not in st.session_state.locations:
        st.session_state.locations.append(new_loc)
        save_locs()
loc_to_del = st.sidebar.selectbox("Delete location", [""]+st.session_state.locations)
if st.sidebar.button("➖ Delete"):
    if loc_to_del:
        st.session_state.locations.remove(loc_to_del)
        save_locs()

# Consolidated Entry Form
st.header("📝 Enter / Edit Item")
with st.form("item_form"):
    name = st.text_input("Item name")
    loc = st.selectbox("Item location", [""] + st.session_state.locations)
    price = st.number_input("Item price", min_value=0.0, step=0.01)
    misc = st.text_area("Miscellaneous")
    pic = st.file_uploader("Upload picture", type=["png","jpg","jpeg"])
    search_btn = st.form_submit_button("🔍 Search or Save")

if search_btn:
    existing = items[items.name.str.lower()==name.lower()]
    if not name:
        st.error("Enter an item name.")
    elif not loc:
        st.error("Select a location.")
    else:
        if not existing.empty:
            idx = existing.index[0]
            items.at[idx, "location"] = loc
            items.at[idx, "price"] = price
            items.at[idx, "misc"] = misc
            st.success(f"Updated '{name}'.")
        else:
            barcode = qrcode.make(name)
            barcode_path = f"./Items/{name}_barcode.png"
            barcode.save(barcode_path, use_container_width=True)
            pic_path = ""
            if pic:
                pic_path = f"./Items/{name}_pic{os.path.splitext(pic.name)[1]}"
                with open(pic_path, "wb") as f: f.write(pic.read())
            items.loc[len(items)] = {
                "name": name, "location": loc, "price": price,
                "misc": misc, "barcode_img": barcode_path,
                "picture_path": pic_path
            }
            st.success(f"Saved new item '{name}'.")
        save_items()

# Listing and actions
st.header("🔎 Item Lookup & Actions")
search = st.text_input("Search item by name")
if st.button("Search"):
    df = items[items.name.str.contains(search, case=False)] if search else items
    st.dataframe(df[["name","location","price","misc"]], use_container_width=True)
    for _, row in df.iterrows():
        st.subheader(row.name)
        col1, col2 = st.columns(2)
        with col1:
            st.image(row.barcode_img, caption="Barcode", use_container_width=True)
            if st.button(f"Print Barcode {row.name}"):
                st.write(f"**Print:** {row.barcode_img}")
        with col2:
            if row.picture_path and os.path.exists(row.picture_path):
                st.image(row.picture_path, caption="Picture", use_container_width=True)
        st.write(f"**Location:** {row.location}")
        st.write(f"**Price:** ${row.price:.2f}")
        st.write(f"**Misc:** {row.misc}")

# Barcode scanner using camera
st.header("📷 Scan Barcode")
img_file = st.camera_input("Scan QR code")
if img_file:
    img = Image.open(img_file)
    arr = cv2.imdecode(np.frombuffer(img_file.getvalue(), np.uint8), cv2.IMREAD_COLOR)
    detector = cv2.QRCodeDetector()
    data, _, _ = detector.detectAndDecode(arr)
    if data:
        st.success(f"Scanned data: {data}")
        # populate form
        found = items[items.name==data]
        if not found.empty:
            r = found.iloc[0]
            st.write("### Found item info:")
            st.write(dict(r[["name","location","price","misc"]]))
        else:
            st.warning("Not found. Enter details above to add.")
    else:
        st.error("No QR detected.")
