# Testing Signup Flow

## Start Dev Server

```bash
cd /Users/Herd/purifymind/apps/web
pnpm dev
```

Dev server will start on: http://localhost:3001

## Test Signup

1. Navigate to: http://localhost:3001/sign-up
2. Enter email and password
3. Click "Sign Up"
4. Check Supabase email for verification link (if email confirmations enabled)

## Verify Profile Creation

After signup, verify the profile was created:

```bash
# Connect to Supabase
PGPASSWORD='3RG68o6NiUHygxq0' psql -h db.emdudnbushwjfcsuoxcj.supabase.co -U postgres -d postgres -p 5432

# Check latest profile
SELECT 
  id, 
  full_name, 
  email,
  referral_code,
  created_at
FROM auth.users
ORDER BY created_at DESC 
LIMIT 1;

# Verify profile was created by trigger
SELECT 
  id,
  full_name,
  avatar_url,
  referral_code,
  onboarding_completed,
  created_at
FROM public.profiles
ORDER BY created_at DESC
LIMIT 1;

# Exit psql
\q
```

## Expected Results

1. **User created in auth.users**: Email, encrypted password
2. **Profile created in public.profiles**: Same ID as auth.users, auto-generated referral_code
3. **Referral code**: 8-character uppercase alphanumeric (e.g., "A3F8D2B1")

## Test Sign-In

1. Navigate to: http://localhost:3001/sign-in
2. Enter email and password
3. Click "Sign In"
4. Should redirect to `/protected` (if configured)

## Common Issues

### Issue: Profile not created
**Symptom**: User exists in auth.users but not in public.profiles
**Fix**: Check trigger exists and has correct search_path
```sql
SELECT tgname, tgenabled FROM pg_trigger WHERE tgname = 'on_auth_user_created';
```

### Issue: Referral code not generated
**Symptom**: Profile created but referral_code is NULL
**Fix**: Check trigger exists
```sql
SELECT tgname FROM pg_trigger WHERE tgname = 'profiles_create_referral_code';
```

### Issue: RLS blocking queries
**Symptom**: Cannot query profiles table
**Fix**: Use service role key or check RLS policies
```sql
SELECT * FROM pg_policies WHERE tablename = 'profiles';
```

## Quick Database Check

```bash
# One-liner to check all tables exist
PGPASSWORD='3RG68o6NiUHygxq0' psql -h db.emdudnbushwjfcsuoxcj.supabase.co -U postgres -d postgres -p 5432 -c "\dt public.*"

# Expected output:
#  Schema |     Name      | Type  |  Owner   
# --------+---------------+-------+----------
#  public | block_events  | table | postgres
#  public | notifications | table | postgres
#  public | partnerships  | table | postgres
#  public | profiles      | table | postgres
#  public | referrals     | table | postgres
```
