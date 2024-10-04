# Spotify_SQL
In this respository, i do SQL window function to solve problem. It'll give insight for the reader about what is in spotify.

CREATE DATABASE spotify;
-- create table
USE spotify;

-- USING PHYTON TO INSERT DATA -- 
 


-- ------ EDA -------------
-- --------------------------
SELECT * FROM spotify;
SELECT count(*) as row_num FROM spotify;
SELECT count(distinct artist) FROM spotify;
SELECT count(distinct album) FROM spotify;
SELECT DISTINCT Album_type FROM spotify;
SELECT ROUND(MIN(duration_min),2)FROM spotify;
SELECT ROUND(MAX(duration_min),2) FROM spotify;
SELECT ROUND(AVG(duration_min),2) FROM spotify;
SELECT DISTINCT channel FROM spotify;
SELECT DISTINCT most_playedon FROM spotify;
SELECT DISTINCT licensed FROM spotify;
SELECT ROUND(MAX(tempo),2) FROM spotify;
SELECT ROUND(MIN(tempo),2) FROM spotify;
SELECT ROUND(AVG(tempo),2) FROM spotify;


DELETE FROM spotify
WHERE duration_min = 0;

SELECT 
	SUM(CASE WHEN licensed = 1 THEN 1 ELSE 0 END) as count_true,
    SUM(CASE WHEN licensed = 0 THEN 1 ELSE 0 END) as count_false
FROM spotify;

SELECT artist, track, tempo FROM spotify
WHERE (SELECT ROUND(MAX(tempo),2) FROM spotify)
ORDER BY 3 desc;



-- OPTIMIZING DATA -------------------------------------
-- -----------------------------------------------------

-- Retrieve the names of all tracks that have more than 1 billion streams.--
SELECT track, stream FROM spotify
WHERE stream > 1000000000
ORDER BY 2 DESC;

-- List all albums along with their respective artists.--
SELECT DISTINCT album, artist FROM spotify
ORDER BY 1;
SELECT DISTINCT album FROM spotify;


-- Get the total number of comments for tracks where licensed = TRUE.---
SELECT SUM(comments) as total_comment 
	FROM spotify
WHERE licensed = 'true'
;


-- Find all tracks that belong to the album type single --
SELECT track
	FROM spotify
WHERE album_type = 'single';

-- Count the total number of tracks by each artist.--
SELECT COUNT(*) as total_track, artist
	FROM spotify
GROUP BY 2
ORDER BY 1 DESC;

-- Calculate the average danceability of tracks in each album.--
SELECT AVG(danceability), track FROM spotify
GROUP BY 2;

-- Find the top 5 tracks with the highest energy values--
SELECT track, energy FROM spotify
	ORDER BY 2 DESC
	LIMIT 5;

-- List all tracks along with their views and likes where official_video = TRUE--
SELECT artist, track, views, likes, official_video FROM spotify
	WHERE official_video = 'true'
    ORDER BY 3 DESC;
    
-- For each album, calculate the total views of all associated tracks.--
SELECT album, track, SUM(views) FROM spotify
	group by 1,2
	ORDER BY 3 DESC;

-- Retrieve the track names that have been streamed on Spotify more than YouTube--
WITH CTE AS
(SELECT track,
	COALESCE(sum(CASE WHEN most_playedon = 'youtube' THEN stream END),0) as stream_yt,
    COALESCE(sum(CASE WHEN most_playedon = 'spotify' THEN stream END),0) as stream_spotify
FROM spotify
GROUP BY 1)
SELECT * FROM CTE
	WHERE stream_spotify > stream_yt
	AND stream_yt <> 0
;


-- Find the top 3 most-viewed tracks for each artist using window functions--
SELECT * FROM
(SELECT artist, track, SUM(views) as total_views, 
	DENSE_RANK () OVER(PARTITION BY artist ORDER BY SUM(views) DESC) AS ranking
FROM spotify
GROUP BY 1, 2
) t1
WHERE ranking <= 3
AND total_views <> 0
;

-- Write a query to find tracks where the liveness score is above the average.--
SELECT track, artist, ROUND(liveness,2) as liveness_score FROM spotify
	WHERE Liveness >
	(SELECT ROUND(AVG(Liveness),2)
FROM spotify) 
	ORDER BY 3 DESC;

-- Use a WITH CTE clause to calculate the difference between the highest and lowest energy values for tracks in each album.--
WITH CTE_ENERGY 
AS
(SELECT album,
	MAX(energy) AS higest_energy,
    MIN(energy) AS lowest_energy
FROM spotify
GROUP BY 1)
SELECT 
	album,
    ROUND(higest_energy - lowest_energy,2) as energy_diff
FROM CTE_ENERGY
WHERE ROUND(higest_energy - lowest_energy,2) <> 0
ORDER BY 2 DESC
;

-- Find tracks where the energy-to-liveness ratio is greater than 1.2
SELECT track, artist, ROUND(energyliveness,2) as energy_liveness
FROM spotify
	WHERE ROUND(energyliveness,2) > 1.2
    ORDER BY 3 DESC;

-- Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions--
SELECT artist, track, likes FROM spotify;

SELECT artist, track, SUM(likes), views
FROM spotify
GROUP BY 1,2,4
ORDER BY 4 DESC;




