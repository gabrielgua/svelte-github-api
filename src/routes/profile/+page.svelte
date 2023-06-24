<script lang="ts">

    type Profile = {
        id: number;
        bio: string;
        name: string;
        login: string;
        avatar_url: string;
    }

    let profile: Profile;   
    let loading: boolean;

    function getProfile() {
        loading = true;
        fetch('https://api.github.com/users/gabrielgua')
            .then(async res => {
                profile = await res.json();
            })
    }



    

</script>

<div class="profile-card">
    {#if !profile}
        <button on:click={getProfile} disabled={loading}> 
            {#if loading}
                <div class="spinner"></div>
            {/if}    
            Get profile
        </button>
    {:else} 
        <img src="{profile.avatar_url}" alt="Profile">
        <div class="profile__info">
            <hgroup class="profile__info">
                <h3>{profile.name}</h3>
                <h4>@{profile.login}</h4>
            </hgroup>
            <p>{profile.bio}</p>
        </div>
    {/if}
        
    
</div>

<style>
    

    .profile-card {
        margin-block-start: 2rem;
        margin-inline: auto;
        border: 1px solid aquamarine;
        box-shadow: 0px 0px 100px 5px black;
        border-radius: .5rem;
        width: min(100% - 2rem, 40rem);
        
        gap: 2rem;
        display: grid;
        grid-template-columns: auto 3fr;
        padding: 2rem;
    }

    .profile-card img {
        max-width: 10rem;
        border-radius: 50%;
        outline: 2px solid white;
        outline-offset: 5px;
    }

    .profile__info {
        display: grid;
        align-items: center;
    }

    .profile__info h3 {
        color: var(--clr-title);
        font-size: x-large;
        font-weight: 500;
    }

    .profile__info h4 {
        color: var(--clr-subtitle);
        font-weight: 400;
    }

    .profile__info p {
        color: var(--clr-text);
        font-weight: 400;
    }

    .profile-card > button {
        border: none;
        background-color: aquamarine;
        color: black;
        padding: 1rem;
        font-size: large;
        font-weight: 600;
        border-radius: .5rem;
        display: grid;
        grid-template-columns: auto 2fr;
        gap: 1rem;
        transition: all 150ms ease;
    }

    .profile-card > button:hover {
        cursor: pointer;
        background-color: rgb(127, 255, 212, .8);
    }

    .profile-card > button:disabled {
        cursor: default;
        background-color: rgb(127, 255, 212, .5);
    }
    

    .spinner {
        border: 3px solid transparent;
        border-inline-end: 3px solid black;
        border-block-start: 3px solid black;
        background-color: transparent;
        width: 2rem;
        height: 2rem;
        border-radius: 50%;

        animation: spin 1000ms ease infinite;
    }

    @keyframes spin {
        100% {
            transform: rotate(360deg);
        }
    }
</style>


