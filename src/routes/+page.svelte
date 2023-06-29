<script lang="ts">
    import { fade, fly } from "svelte/transition";
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
    
    let username = "";
    let userProfile: Profile;
    let userRepos: Repo[] = [];

    let loading: boolean = false;

    const getUser = async () => {

        if (username.length) {
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
    }

    const getRepos = async () => {
        loading = true;
        userRepos = [];
        const fetchRepos = await fetch(`https://api.github.com/users/${username}/repos?per_page=${REPOS_PER_PAGE}&sort=pushed`)
        const repos: Repo[] = await fetchRepos.json();   
        
        
        for await (const repo of repos) {

            const fetchLanguages = await fetch(`https://api.github.com/repos/${username}/${repo.name}/languages`)
            if (fetchLanguages.ok) {
                const langs = await fetchLanguages.json();
                const languages: string[] = Object.keys(langs);
                repo.languages = languages;
            }
        } 

        loading = false;
        for (const repo of repos) {
            userRepos = [...userRepos, repo];
        }
    }
    
   

  

</script>

<div class="container">
    <p class="title">Github profile</p>

    <form on:submit|preventDefault={getUser}>
        <input class="search__input" type="text" name="username" bind:value={username} placeholder="🔍 Search with github username" />
        <button class="search__btn" type="submit">Search</button>
    </form>            

    {#if errorMessage != ""} 
        <p>{errorMessage}</p>
    {/if}

    {#if userProfile && userProfile.name != ''}
        <div class="profile" in:fly={{x: 100}}>
            <img src="{userProfile.avatar_url}" alt="Avatar url">
            <a href="https://github.com/{username}" target="_blank" class="profile__info">
                <p class="profile__name">{userProfile.name}</p>
                <p class="profile__username">@{userProfile.login}</p>
            </a>
            <!-- <p class="profile__bio">{userProfile.bio ? userProfile.bio : 'No profile description 😿.'}</p> -->
        </div>


        <div class="repos">
            {#if loading}
                <p class="loading">loading repos...</p>
            {:else} 
                <p class="repos__title" in:fly={{x: 30}}>Most recently pushed <span>{userProfile.login}</span>'s repositories</p>
            {/if}
            {#each userRepos as repo, i}
                <a href="{repo.html_url}" target="_blank" class="repo" in:fly={{delay: 100 * i, x: -100}}>
                    <div class="repo__info">
                        <div class="ball"></div>
                        <p>{repo.name}</p>
                    </div>
                    <p class="repo__desc multiline-truncate">{repo.description != null ? repo.description : 'No description. 😿'}</p>
                    <div class="repo__langs">
                        {#each repo.languages as lang}
                            <Tag>{lang}</Tag>
                        {/each}
                    </div>
                    <div class="repo__dates">
                        <div class="pushed_date">Last pushed on {new Date(repo.pushed_at).toLocaleDateString()}</div>
                    </div>
                </a>
            {/each}
        </div>


        
    {/if}
</div>

<style>
    .container {
        display: grid;
        place-items: center;
        padding: 1rem;
        padding-block-start: 2rem;
        gap: 1rem;
        width: min(100% - 2rem, 45rem);
        margin-inline: auto;
    }

    .container .title {
        font-size: var(--fs-xx-large);
        place-self: start;
    }

    form {
        width: 100%;
        display: grid;
        grid-template-columns: 1fr auto;
        border: 1px solid var(--clr-border-subtle);
        border-radius: .5rem;

    }

    form input {
        width: 100%;
        /* border: 1px solid var(--clr-border-subtle); */
        padding: 1rem;
        border: none;
        background-color: transparent;
        border-radius: .5rem;
        outline: none;
        transition: outline 50ms ease;
        z-index: 2;
    }

    form input:focus {
        outline: 3px solid var(--clr-primary);
    }

    form button {
        border: none;
        border-radius: 0 .5rem .5rem 0;
        padding: 1rem;
        background-color: transparent;
        color: var(--clr-primary);
        transition: background-color 150ms ease;
        z-index: 1;
    }


    form button:hover { 
        cursor: pointer;
        background-color: black;        
    }

    form button:focus {
        outline-color: var(--clr-primary);

    }

    .profile {
        display: grid;
        align-items: center;
        grid-template-columns: 1fr 5fr;
        gap: 1rem;
        color: var(--clr-title);
        padding-block-end: 1rem;
        border-block-end: 1px solid var(--clr-border-subtle);
    }

    .profile__info {
        text-decoration: none;
        color: inherit;
    }

    .profile__info:hover .profile__name {
        text-decoration: underline;
    }

    .profile__username {
        color: var(--clr-subtitle);
        font-size: var(--fs-small);
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
        border-radius: .5rem;
        color: var(--clr-subtitle);
        font-size: var(--fs-normal);
        background-color: black;
        box-shadow: rgba(0, 0, 0, .5) 0px 7px 29px 0px;
        text-decoration: none;
    }


    .repo:hover .repo__info > p {
        text-decoration: underline;
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

    .repo__info p {
        color: var(--clr-accent);
        font-size: var(--fs-large);
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

    .loading {
        grid-column: span 2;
        place-self: center;
        color: var(--clr-subtitle);
    }

    @media (width <= 550px) {
        .profile {
            grid-template-columns: 1fr;
            place-items: center;
        }

        .profile img {
            max-width: 15rem;
        }

        

        .repos__title {
            grid-column: span 1;
        }
    }
</style>
