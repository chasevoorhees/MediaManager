# WeebAnimeManager
Bookmarking/Listing/Organizing/Running-Episodes Tracking/Linking/Checking



WEEB ANIME MANAGER

Bookmarking/Listing/Organizing (folders/subfolders)


WatchList: Full.Genre[], Running.Genre[], Future.Genre[]
WatchedList: GOAT.Genre[], Good.Genre[], Bad.Genre[], ReWatch.Genre[]
MaybeList: Full.Genre[], Running.Genre[], Future.Genre[]
DontWatchList: Bad.Genre[]

Just Bookmarks? No. 

==================
SETTINGS
==================


Set Genres[]: Drama, Isekai, Slice of Life, etc
-Will be copied into all *List.*.Genre[].ANIME

-Set Preferred Streaming Site(s) eg. animeseries, animepahe
	=PST
	
-Set per Anime episode string and count type per PST
	=PST.Anime.EpisodeString = string (eg "Log Horizon")
	=PST.Anime.EpisodeCountType = decimal, string (eg 1,2,3 or one,two,three)
	
	
==================
FEATURES
==================


WatchList - Features:
	Current Episode # Tracking
	Play Next
	Manual Move Anime to Folder/SubFolder
	

WatchedList - Features:
	Above Features
	More TBD
	

MaybeList - Series Running - Features:
	Pulls Release Day/Time from MAL
	Auto-Update/Notify on new episode available
	Checks PST either X (user chosen) or 1-2-4-6-12hrs after MAL release time.
	Notifies on Episode Available
	Series Completed Notifications, Moves to Series Complete


WatchList - Series TBR - Features:
	Pulls Release Date from MAL
	Notifies on Release Date
	Notifies on Episodes Available, Moves to Series Running


==================
DATA STRUCTURE
==================


PST.
	WatchType[Watch, Watched, MaybeWatch, DontWatch].
		ReleaseType[Full, Running, Future, GOAT, Good, Bad, ReWatch].
			Genre[Drama, Isekai, Slice of Life, etc].
				Anime[ID, ReleaseURL, EpisodeString, EpisodeCountType, CurrentEpisodeWatched, CurrentEpisodeReleased, MALUrl, ReleaseDateTime, ReleaseCheckTypeHours[], FutureBool, NotifyNewEpisode, CompleteBool, NotifyCompleteBool]

==================


class InitPST{
		
	PST pst = new PST('http://animeseries.io', new Array['Watch', 'Watched', 'MaybeWatch', 'DontWatch']);
	
	
}


class PST{
	url;
	watchTypeList = {};
	constructor(url, watchTypes) { //'Watch', 'Watched', 'MaybeWatch', 'DontWatch'
		foreach (str in watchTypes){
			this.watchTypeList[str] = new WatchType(new Array['Full', 'Running', 'Future', 'GOAT', 'Good', 'Bad', 'ReWatch']);
		}
		this.watchTypeList.OrderBy(s=>s.name);
		this.url = url;
   }
}

class WatchType{
	releaseTypeList = {};
	constructor(releaseTypeStrings) { //'Full', 'Running', 'Future', 'GOAT', 'Good', 'Bad', 'ReWatch'
		foreach (str in releaseTypeStrings){
			this.releaseTypeList[str] = new Genre(new Array['Drama', 'Isekai', 'Slice of Life']);
		
			this.releaseTypeList.OrderBy(s=>s.key);
		}
    }
}

class Genre{
	genreList = {};
	constructor(genreStrings) { //Drama, Isekai, Slice of Life, etc
		
		foreach (str in genreStrings){
			this.genreList[str] = {};//Anime
		}
		this.genreList.OrderBy(s=>s.key);
    }
	
	addAnime(genre, name, anime){
		this.genreList[genre][name] = anime;
		return;
	}
/*	
	addAnimeList(genre, animeList){
		var tempGenreList = {};
		foreach (a in animeList){
			this.tempGenreList[genre][a.name] = a//Anime
		}
		this.tempGenreList.OrderBy(s=>s.key);	
		
		this.tempGenreList[genre][name] = anime;
		return;
	}
*/
	removeAnime(genre, name, anime){
		var ctn = this.genreList[genre].Contains(name);
		if (ctn){
			this.genreList[genre].Remove(name);
			return true;
		}
		return false;
	}
}

class Episode {
	url;
	episodeNumber;
	constructor(url, num){
		this.url = url;
		this.episodeNumber = num;
	}
}
class Anime {
	id;
	name;
	releaseURL;
	episodeString;
	episodeCountTypeDigit;
	currentEpisodeWatched;
	malUrl;
	runningCheckXHoursList[];
	futureBool;
	futureNotifyBool;
	futureReleaseDateTime;
	futureReleaseDateTimeFoundBool;
	runningBool;
	runningReleaseDateTime;
	runningReleaseDateTimeFoundBool;
	runningNotifyWhenNewEpisodeNewBool;
	runningNotifyNewEpisodeBool;
	
	unwatchedEpisodes[]; //Episode
	watchedEpisodes[]; //Episode
	allEpisodes[]; //Episode
	lastEpisodeWatched; //Episode
	lastEpisodeReleased; //Episode

	completeBool;
	completeNotifyBool;
	
	constructor(id, name, releaseURL, episodeString, episodeCountTypeDigit, lastEpisodeWatched, malUrl, runningCheckXHoursList, runningNotifyWhenNewEpisodeNewBool, runningBool, futureBool, completeBool, completeNotifyBool) {
		this.id = id;
		this.name = name;
		this.releaseURL = releaseURL;
		this.episodeString = episodeString;
		this.episodeCountTypeDigit = episodeCountTypeDigit;
		this.lastEpisodeWatched = lastEpisodeWatched;		
		this.malUrl = malUrl;
		this.futureBool = futureBool;
		this.runningBool = runningBool;	
		this.completeBool = completeBool;
		this.runningCheckXHoursList[] = runningCheckXHoursList;
		this.runningNotifyOnNewBool = runningNotifyOnNewBool;
		this.completeNotifyOnEndBool = completeNotifyBool;

		
		switch (true) {
		  case runningBool:
			this.runningReleaseDateTime = findMALRunningReleaseDateTime(malUrl, runningBool, runningReleaseDateTimeFoundBool);
			
			updateRunningEpisodes(this);

			break;
		  
		  case futureBool:
			this.futureReleaseDateTime = findMALFutureReleaseDateTime(malUrl, futureBool,futureReleaseDateTimeFoundBool);	  
			break;
		  
		  case completeBool:
			updateCompleteEpisodes(this);
			
			break;
		  
		  default:
			console.log(`Sorry, we are out of ${expr}.`);
		}
		
		
		


		
		
		
   }
  
  updateRunningEpisodes(this) {
	unwatchedEpisodes[]; //Episode
	watchedEpisodes[]; //Episode
	allEpisodes[]; //Episode
	lastEpisodeWatched; //Episode
	lastEpisodeReleased; //Episode
	new Episode(url, num);
	
	
	  
	//check still running
		//this.runningNotifyNewEpisodeBool
		//this.runningBool
	
	
	//get latest episodes
	if (this.runningBool){
		//this.runningNotifyNewEpisodeBool = checkNewEpisodesAvailable(this.releaseURL, this.episodeString, this.episodeCountTypeDigit, this.runningLastEpisodeWatched, this.runningLastEpisodeReleased, this.runningWatchedEpisodes, this.runningUnwatchedEpisodes, this.runningAllEpisodes);
		this.runningNotifyNewEpisodeBool = checkNewEpisodesAvailable(this);
	}
	if (!this.runningNotifyWhenNewEpisodeNewBool){
		this.runningNotifyNewEpisodeBool = false;
	}
	
    return this.runningNotifyNewEpisodeBool;
  }

  checkNewEpisodesAvailable(this){
	  
	  //got to.. call out to PST/releaseURL, find next episode string/int, search for epi str + next string/int, return link
	  
	  //releaseURL, episodeString, episodeCountTypeDigit, runningLastEpisodeWatched, runningLastEpisodeReleased, runningWatchedEpisodes, runningUnwatchedEpisodes, runningAllEpisodes
	  
	  return;
	  
	  
  }
  
  
  updateCompleteEpisodes(this) {
	unwatchedEpisodes[]; //Episode
	watchedEpisodes[]; //Episode
	allEpisodes[]; //Episode
	lastEpisodeWatched; //Episode
	lastEpisodeReleased; //Episode
	new Episode(url, num);
	
	allEpisodes
	  
	//check still running
		//this.runningNotifyNewEpisodeBool
		//this.runningBool
	
	
	//get latest episodes
	if (this.runningBool){
		//this.runningNotifyNewEpisodeBool = checkNewEpisodesAvailable(this.releaseURL, this.episodeString, this.episodeCountTypeDigit, this.runningLastEpisodeWatched, this.runningLastEpisodeReleased, this.runningWatchedEpisodes, this.runningUnwatchedEpisodes, this.runningAllEpisodes);
		this.runningNotifyNewEpisodeBool = checkNewEpisodesAvailable(this);
	}
	if (!this.runningNotifyWhenNewEpisodeNewBool){
		this.runningNotifyNewEpisodeBool = false;
	}
	
    return this.runningNotifyNewEpisodeBool;
  }
  
  
  findMALRunningReleaseDateTime(malUrl, runningBool, runningReleaseDateTimeFoundBool){
	var dt;
	if (runningBool){	
		if (dt = getMalRunningRTD(malUrl) != ""){
			this.runningReleaseDateTime = dt;
			this.runningReleaseDateTimeFoundBool = true;
			return dt;
		}else{
			this.runningReleaseDateTime = "";
			this.runningReleaseDateTimeFoundBool = false;
			return "";
		}
	}else{
		this.runningReleaseDateTime = "";
		this.runningReleaseDateTimeFoundBool = false;
		return "";
	}  
  }
  
  findMALFutureReleaseDateTime(malUrl, futureBool, futureReleaseDateTimeFoundBool){
	var dt;
	if (futureBool){	
		if (dt = getMalFutureRTD(malUrl) != ""){
			this.futureReleaseDateTime = dt;
			this.futureReleaseDateTimeFoundBool = true;
			return dt;
		}else{
			this.futureReleaseDateTime = "";
			this.futureReleaseDateTimeFoundBool = false;
			return "";
		}
	}else{
		this.futureReleaseDateTime = "";
		this.futureReleaseDateTimeFoundBool = false;
		return "";
	}  
  }
  
  getMalRunningRTD(malUrl){
	  
	  
  }
  
  getMalFutureRTD(malUrl){
	  
	  
  }
}




======================================================
======================================================



public class PST{
	public string URL = 'https://animeseries.io";
	WatchType[] watchTypes = {'Watch', 'Watched', 'MaybeWatch', 'DontWatch'};

}


public class WatchType{
	ReleaseType[] releaseTypes = {};

}


var animes = {
	id: 1
	releaseURL: "https://animeseries.io/anime"
	name: "Log Horizon",
	episodeString: "Horizon",
	episodeCountType: "one" "1",
	CurrentEpisodeWatched: "Horizon",
	CurrentEpisodeReleased: "Horizon",
	MALUrl: "Horizon",
	ReleaseDateTime: "Horizon",
	ReleaseCheckTypeHours: "1,2,3,6",
	FutureAnimeBool: "false",
	NotifyNewEpisode: "true",
	CompletedAnimeBool: "false",
	NotifyCompletedAnimeBool: "false"			
}
animes["key"] = "val";

Drama, Isekai, Slice of Life, etc

