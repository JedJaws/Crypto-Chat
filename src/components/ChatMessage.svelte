<script>
    export let messageText;
    export let messageTime;
    export let messageUser;
</script>

<div class="message">
    <div class="message-user">
        {messageUser}:
    </div>
    <div class="message-text">
        {messageText}
    </div>
    <div class="message-time">
        {messageTime}
    </div>
</div>

<style>
    .message {
        display: flex;
        flex-direction: row;
        align-items: flex-start;
        margin: 10px 0;
        color: white
    }
    
    .message *{
        margin-left: 10px;
    }
    .message-user {
        font-weight: bold;
    }

    .message-time {
        font-size: 0.8rem;
        color: #999;
    }
</style>