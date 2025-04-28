# == alpha.py ==
from fastapi import FastAPI, Response, Query
from telethon import TelegramClient
from telethon.tl.functions.contacts import GetContactsRequest
from telethon.tl.types import UserStatusOnline, UserStatusOffline
import asyncio
import os
import re
import io
import phonenumbers
from phonenumbers import carrier

# Telethon config
api_id = 9418515
api_hash = 'f1dc0dcb11400f317694762bbde5c801'
session_name = 'vercel_session'

client = TelegramClient(session_name, api_id, api_hash)

app = FastAPI()

def is_id_cantik(user_id):
    id_str = str(user_id)
    return (
        re.fullmatch(r'(\d)\1{3,}', id_str) or
        id_str in ['111', '222', '333', '444', '555', '666', '777', '888', '999',
                   '1111', '2222', '3333', '4444', '5555', '6666', '7777', '8888', '9999',
                   '12345', '4321', '54321']
    )

def get_operator_name(phone):
    try:
        if not phone.startswith('+'):
            phone = '+' + phone
        number = phonenumbers.parse(phone, None)
        operator_name = carrier.name_for_number(number, 'en')
        return operator_name if operator_name else "Unknown"
    except Exception:
        return "Unknown"

async def delete_session_file():
    for ext in ['', '.session', '.session-journal']:
        file_path = f"{session_name}{ext}"
        if os.path.exists(file_path):
            os.remove(file_path)

async def generate_output(users, page=1, per_page=100):
    start = (page - 1) * per_page
    end = start + per_page
    users_page = users[start:end]

    if not users_page:
        return "📋 Tidak ada kontak pada halaman ini."

    output = f"📋 Halaman {page} - Menampilkan {len(users_page)} kontak\n\n"

    for i, u in enumerate(users_page, 1):
        nama = u.first_name or ""
        username = f"@{u.username}" if u.username else ""
        phone = u.phone or "Nomor tidak tersedia"
        operator = get_operator_name(phone)
        id_cantik = " ✅ ID CANTIK" if is_id_cantik(u.id) else ""
        
        if isinstance(u.status, UserStatusOnline):
            status = "sedang online"
        elif isinstance(u.status, UserStatusOffline):
            last = u.status.was_online.strftime("%Y-%m-%d %H:%M:%S%z") if u.status and u.status.was_online else "Tidak diketahui"
            status = f"offline (terakhir: {last})"
        else:
            status = "baru-baru ini / tidak diketahui"

        output += (
            f"💥 Kontak {i}\n"
            f"  • ID: {u.id}{id_cantik}\n"
            f"  • Nama: {nama}\n"
            f"  • Username: {username}\n"
            f"  • Nomor: {phone} ({operator})\n"
            f"  • Status: {status}\n\n"
        )

    return output

@app.get("/api/generate-contacts")
async def generate_contacts(page: int = Query(1, ge=1)):
    await client.connect()
    if not await client.is_user_authorized():
        await client.disconnect()
        await delete_session_file()
        return {"error": "Session belum login."}

    result = await client(GetContactsRequest(hash=0))
    users = result.users

    output = await generate_output(users, page=page)

    await client.disconnect()
    await delete_session_file()
    
    return {"result": output}

@app.get("/api/download-contacts")
async def download_contacts():
    await client.connect()
    if not await client.is_user_authorized():
        await client.disconnect()
        await delete_session_file()
        return {"error": "Session belum login."}

    result = await client(GetContactsRequest(hash=0))
    users = result.users

    output = await generate_output(users, page=1, per_page=len(users))  # Semua kontak

    await client.disconnect()
    await delete_session_file()

    return Response(
        content=output,
        media_type="text/plain",
        headers={"Content-Disposition": "attachment; filename=contacts.txt"}
    )
