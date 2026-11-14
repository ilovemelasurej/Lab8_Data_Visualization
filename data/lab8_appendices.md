import os
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from matplotlib.animation import FuncAnimation, PillowWriter
import webbrowser

df = pd.read_csv("spotify_top_1000_tracks.csv")

student_name = "Roel Jr. Ropal Jerusalem"
student_id = "2026-BSECE-1C"
color_bar = "indianred"
color_line = "royalblue"
cmap_color = "coolwarm"

longest_tracks = df.nlargest(10, 'duration_min')
display(longest_tracks[['track_name', 'artist', 'duration_min']])

plt.figure(figsize=(12, 6))
plt.barh(longest_tracks['track_name'], longest_tracks['duration_min'], color=color_bar, edgecolor='darkred')
plt.title(f"{student_name} ({student_id})\nTop 10 Longest Tracks", fontsize=14)
plt.xlabel('Duration (minutes)')
plt.ylabel('Track Name')
plt.gca().invert_yaxis()
plt.grid(True, linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

df['release_date'] = pd.to_datetime(df['release_date'], errors='coerce')
df['year'] = df['release_date'].dt.year
yearly_counts = df.groupby('year').size().reset_index(name='count')
yearly_counts['cumulative'] = yearly_counts['count'].cumsum()
yearly_counts = yearly_counts.sort_values('year')

fig, ax1 = plt.subplots(figsize=(12, 6))
ax1.bar(yearly_counts['year'], yearly_counts['count'], color=color_bar, alpha=0.7, label='Yearly Count')
ax1.set_xlabel('Release Year', fontsize=12)
ax1.set_ylabel('Number of Songs', color=color_bar, fontsize=12)
ax1.tick_params(axis='y', labelcolor=color_bar)

ax2 = ax1.twinx()
ax2.plot(yearly_counts['year'], yearly_counts['cumulative'], color=color_line, linewidth=2.5, marker='o', label='Cumulative Growth')
ax2.set_ylabel('Cumulative Total', color=color_line, fontsize=12)
ax2.tick_params(axis='y', labelcolor=color_line)

plt.title(f"{student_name} ({student_id})\nSong Release Trend and Cumulative Growth", fontsize=14)
plt.grid(True, linestyle='--', alpha=0.5)
plt.tight_layout()
plt.show()

df_anim = df.dropna(subset=['year', 'duration_min', 'popularity']).copy()
df_anim['year'] = df_anim['year'].astype(int)
years_sorted = sorted(df_anim['year'].unique())

fig_anim, ax_anim = plt.subplots(figsize=(10, 6))

def animate_hexbin(i):
    ax_anim.clear()
    current_year = years_sorted[i]
    subset = df_anim[df_anim['year'] <= current_year]
    hb = ax_anim.hexbin(subset['duration_min'], subset['popularity'], gridsize=20, cmap=cmap_color, mincnt=1, alpha=0.8)
    ax_anim.set_xlim(df_anim['duration_min'].min() - 0.5, df_anim['duration_min'].max() + 0.5)
    ax_anim.set_ylim(-5, 105)
    ax_anim.set_title(f"{student_name} ({student_id})\nAnimated Hexbin Density (Up to Year: {current_year})", fontsize=14)
    ax_anim.set_xlabel("Duration (minutes)")
    ax_anim.set_ylabel("Popularity Score")
    return hb,

anim = FuncAnimation(fig_anim, animate_hexbin, frames=len(years_sorted), interval=300, repeat=False)
gif_path = os.path.abspath("animated_hexbin_density.gif")
anim.save(gif_path, writer=PillowWriter(fps=5))
print(f"GIF saved successfully at: {gif_path}")
webbrowser.open(f"file://{gif_path}")
plt.close(fig_anim)