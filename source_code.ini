import os
import sys
import logging
from urllib.parse import urlparse, parse_qs, urlencode, urlunparse
import yt_dlp
LOG_FILE = 'spectra_downloader.log'
logging.basicConfig(filename=LOG_FILE, filemode='a', format='%(asctime)s - %(levelname)s - %(message)s', level=logging.INFO)

def clean_url(url):
    parsed = urlparse(url.strip())
    if parsed.netloc in ['youtu.be', 'www.youtu.be']:
        video_id = parsed.path.lstrip('/')
        if video_id:
            return f'https://www.youtube.com/watch?v={video_id}'
        return url
    query = parse_qs(parsed.query)
    video_id = query.get('v', [None])[0]
    if video_id:
        clean_query = urlencode({'v': video_id})
        clean_parsed = parsed._replace(query=clean_query)
        return urlunparse(clean_parsed)
    return url

def progress_hook(d):
    if d['status'] == 'downloading':
        percent = d.get('_percent_str', '').strip()
        speed = d.get('_speed_str', '').strip()
        eta = d.get('_eta_str', '').strip()
        print(f'\r[spectra] Downloading... {percent} at {speed}, ETA: {eta}', end='', flush=True)
    elif d['status'] == 'finished':
        print('\n[spectra] Download finished, processing...')

def list_formats(url):
    ydl_opts = {'quiet': True, 'no_warnings': True}
    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(url, download=False)
            formats = info.get('formats', [])
            print(f"\n[spectra] Available progressive formats for '{info.get('title', 'unknown')}':\n")
            progressive_formats = [f for f in formats if f.get('acodec') != 'none' and f.get('vcodec') != 'none']
            for f in progressive_formats:
                ext = f.get('ext', '')
                fmt_id = f.get('format_id', '')
                resolution = f.get('resolution') or f.get('abr')
                filesize = f.get('filesize') or f.get('filesize_approx')
                filesize_mb = filesize / 1024 / 1024 if filesize else 'N/A'
                print(f"  Format ID: {fmt_id}\tType: {ext}\tResolution/Bitrate: {resolution}\tSize: {(filesize_mb if isinstance(filesize_mb, str) else f'{filesize_mb:.2f} MB')}")
            print()
            return progressive_formats
    except Exception as e:
        print(f'[spectra] Failed to retrieve format list: {e}')
        return []

def get_format_choice(formats):
    valid_ids = {f['format_id'] for f in formats}
    while True:
        choice = input('[spectra] Enter the format ID to download (or press Enter for default best): ').strip()
        if choice == '':
            return
        if choice in valid_ids:
            return choice
        print('[spectra] Invalid format ID. Please try again.')

def download_video(url, output_dir, format_id=None, download_playlist=False):
    ydl_opts = {'outtmpl': os.path.join(output_dir, '%(playlist_title)s/%(title)s.%(ext)s') if download_playlist else os.path.join(output_dir, '%(title)s.%(ext)s'), 'progress_hooks': [progress_hook], 'quiet': True, 'no_warnings': True, 'noplaylist': not download_playlist, 'format': format_id or 'best[ext=mp4]/best'}
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        ydl.download([url])

def main():
    download_dir = os.path.join(os.getcwd(), 'DownloadedSpectraVideos')
    os.makedirs(download_dir, exist_ok=True)
    max_retries = 3
    retries = 0
    while retries < max_retries:
        try:
            url = input('[spectra] Enter the YouTube video or playlist URL: ').strip()
            cleaned_url = clean_url(url)
            print(f'[spectra] Cleaned URL: {cleaned_url}')
            if not cleaned_url.startswith('https://www.youtube.com/watch?v=') and (not cleaned_url.startswith('https://www.youtube.com/playlist?')):
                print('[spectra] The URL does not seem to be a valid YouTube video or playlist URL.')
                retries += 1
                continue
            download_playlist = False
            if 'list=' in cleaned_url or 'playlist?' in cleaned_url:
                playlist_choice = input('[spectra] This looks like a playlist URL. Download entire playlist? (y/n): ').strip().lower()
                download_playlist = playlist_choice == 'y'
            formats = list_formats(cleaned_url)
            format_choice = get_format_choice(formats)
            print('[spectra] Download will start shortly...\n')
            download_video(cleaned_url, download_dir, format_choice, download_playlist)
            print(f'\n[spectra] Download completed successfully. Files saved to: {download_dir}')
            logging.info(f"Downloaded: {cleaned_url} to {download_dir} with format {format_choice or 'default'}")
            return
        except KeyboardInterrupt:
            print('\n[spectra] Download interrupted by user. Exiting.')
            sys.exit(0)
        except Exception as e:
            print(f'\n[spectra] Error: {e}')
            logging.error(f"Error downloading {(cleaned_url if 'cleaned_url' in locals() else url)}: {e}")
            retries += 1
            if retries < max_retries:
                print(f'[spectra] Retry attempt {retries} of {max_retries}...\n')
            else:
                print('[spectra] Maximum retries reached. Exiting.')
                break
if __name__ == '__main__':
    main()
