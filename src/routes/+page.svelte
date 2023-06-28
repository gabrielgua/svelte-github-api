<script lang="ts">
    import { fly } from "svelte/transition";
	import Tag from "$lib/Tag.svelte";

    const REPOS_PER_PAGE = 6

    class Profile {
        name: string = '';
        login: string = '';
        bio: string = '';
        avatar_url: string = '';
    }

    interface Repo {
        name: string;
        description: string;
        html_url: string;
        languages: string[];
        created_at: Date;
        pushed_at: Date;
    } 

    let errorMessage = "";
    
    let username = "gabrielgua";
    let userProfile: Profile;
    let userRepos: Repo[] = [];

    const getUser = async () => {
        userProfile = new Profile();
        errorMessage = "";
        const fetchUser = await fetch(`https://api.github.com/users/${username}`)
        if (!fetchUser.ok) {
            errorMessage = `User '${username}' not found.`;  
            return;       
        }
        
        userProfile = await fetchUser.json();
        getRepos();
    }

    const getRepos = async () => {
        userRepos = [];
        const fetchRepos = await fetch(`https://api.github.com/users/${username}/repos?per_page=${REPOS_PER_PAGE}&sort=pushed`)
        const repos: Repo[] = await fetchRepos.json();   
          
          
        repos.forEach(async repo => {
            
            const fetchLanguages = await fetch(`https://api.github.com/repos/${username}/${repo.name}/languages`)
            if (fetchLanguages.ok) {
                const langs = await fetchLanguages.json();
                const languages: string[] = Object.keys(langs);
                repo.languages = languages;
            }

            userRepos = [...userRepos, repo];
            
        })

        

    }
    
   

  

</script>

<div class="container">
    <p class="title">Github profile fetcher</p>

    <form method="GET" on:submit|preventDefault={getUser}>
        <input class="search__input" type="text" name="username" bind:value={username} />
        <button class="search__btn" type="submit">Search</button>
    </form>            

    {#if errorMessage != ""} 
        <p>{errorMessage}</p>
    {/if}

    {#if userProfile && userProfile.name != ''}
        <div class="profile" in:fly={{x: 100}}>
            <img src="{userProfile.avatar_url}" alt="Avatar url">
            <div class="profile__info">
                <p class="profile__name">{userProfile.name}</p>
                <p class="profile__username">@{userProfile.login}</p>
            </div>
            <p class="profile__bio">{userProfile.bio}</p>
        </div>


        <div class="repos">
            <p class="repos__title" in:fly={{x: 30}}>Most recently pushed <span>{userProfile.login}</span>'s repositories</p>
            {#each userRepos as repo, i}
                <div class="repo" in:fly={{delay: 100 * i, x: -100}}>
                    <div class="repo__info">
                        <div class="ball"></div>
                        <a href="{repo.html_url}" target="_blank" class="singleline-truncate">{repo.name}</a>
                    </div>
                    <p class="repo__desc multiline-truncate">{repo.description != null ? repo.description : 'No description.'}</p>
                    <div class="repo__langs">
                        {#each repo.languages as lang}
                            <Tag>{lang}</Tag>
                        {/each}
                    </div>
                    <div class="repo__dates">
                        <div class="pushed_date">Last pushed on {new Date(repo.pushed_at).toLocaleDateString()}</div>
                    </div>
                </div>
            {/each}
        </div>
        
    {/if}
</div>

<style>
    .container {
        display: grid;
        place-items: center;
        padding: 3rem;
        gap: 1rem;
        width: min(100% - 2rem, 45rem);
        margin-inline: auto;
    }

    .container .title {
        font-size: var(--fs-xx-large);
        text-transform: uppercase;
    }

    .profile {
        border: 1px solid var(--clr-border-subtle);
        display: grid;
        align-items: center;
        grid-template-columns: 1fr 5fr;
        gap: 1rem;
        color: var(--clr-title);
        padding: 1.5rem;
        border-radius: 0 1rem;
    }

    .profile__username {
        color: var(--clr-subtitle);
    }

    .profile__bio {
        grid-column: span 2;
        color: var(--clr-text);
    }

    .profile img {
        width: 100%;
        border-radius: 50%;
    }

    .repos {
        display: grid;
        gap: 1rem;
        grid-template-columns: repeat(auto-fit, minmax(15rem, 1fr));
        width: 100%;
    }

    .repos__title {
        place-self: start;
        grid-column: span 2;

    }

    .repos__title span {
        color: var(--clr-primary);
    }

    .repo {
        border: 1px solid var(--clr-border-subtle);
        padding: .75rem 1rem 1rem;
        display: flex;
        flex-direction: column;
        gap: .5rem;
        border-radius: .5rem 0rem;
        color: var(--clr-subtitle);
        font-size: var(--fs-normal);
    }

    .repo__info {
        display: flex;
        align-items: center;
        gap: .5rem;
    }

    .repo__info .ball {
        width: .75rem;
        height: .75rem;
        background-color: var(--clr-primary);
        border-radius: 50%;
    }

    .repo__info a {
        text-decoration: none;
        color: var(--clr-accent);
        font-size: var(--fs-medium);
    }

    .repo a:hover {
        text-decoration: underline;
    }

    .repo__langs {
        margin-top: auto;
        gap: .25rem;
        display: flex;
        flex-wrap: wrap;
    }

    .repo__dates {
        font-size: var(--fs-small);
    }

    .multiline-truncate {
        overflow: hidden;
        display: -webkit-box;
        -webkit-line-clamp: 3;
        -webkit-box-orient: vertical;  
    }

    .singleline-truncate {
        white-space: nowrap;
        overflow: hidden;
        display: block;
        text-overflow: ellipsis;
    }
</style>
